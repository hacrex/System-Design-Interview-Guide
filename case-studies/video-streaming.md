# Case Study: Video Streaming

Try it yourself first using the [blank template](../templates/design-template.md). Then compare.

## 1. Requirements

**Functional**
- A user can upload a video
- A user can stream (watch) a video on demand
- Adaptive bitrate streaming: client picks quality based on bandwidth
- View count and basic analytics

**Non-functional**
- Video playback must start within ~2 seconds at p95
- High availability: popular videos must always be playable
- Scalable storage and bandwidth: video is large and read-heavy
- Support many concurrent viewers for popular (viral) content

Out of scope for this round: live streaming, DRM/content protection, user accounts/subscriptions, comments, recommendations.

## 2. Estimation

- 10 million DAU, each user watches 3 videos per day
- 30M video plays/day → ~350 plays/s average, ~1,500/s peak
- Uploads: 100K videos/day → ~1.2 uploads/s average, ~5/s peak
- Average video: 10 minutes, ~500 MB after encoding at multiple bitrates (combined ladder ~1.5 GB total storage per video)

Storage:
- 100K × 1.5 GB = 150 TB/day — too much; re-think. Most videos are not watched often.
- Realistic: active catalog 1 million videos × 1.5 GB = 1.5 PB; long tail to cold storage
- Bandwidth: 350 plays/s × ~2 MB/s average bitrate = ~700 MB/s sustained, spikes higher for viral content

Conclusion: storage is in the petabyte range and bandwidth is the dominant cost. Use object storage + CDN for hot content; encode into multiple bitrates; store cold tier for rarely-watched videos.

## 3. API

```
POST   /v1/uploads
       body: { "title": "...", "description": "optional", "size_bytes": "..." }
       returns: { "upload_id": "...", "upload_url": "...", "video_id": "..." }

PUT    /v1/uploads/{upload_id}/parts/{part_number}   (direct to object storage)
       returns: 200 OK

POST   /v1/uploads/{upload_id}/complete
       body: { "parts": [...] }
       returns: { "video_id": "...", "status": "processing" }

GET    /v1/videos/{video_id}/manifest                (HLS/DASH manifest)
       returns: manifest with variant bitrates and segment URLs

GET    /v1/videos/{video_id}/segments/{segment_id}   (video chunk, via CDN)
       returns: video bytes
```

Upload uses direct-to-object-storage (presigned URL) so media bytes never pass through app servers.

## 4. Data model

Access patterns:
- Insert video metadata on upload (write-light)
- Fetch manifest and metadata for playback (read-heavy)
- Increment view count (write-heavy, but can be eventually consistent)

| Entity | Field | Notes |
|--------|-------|-------|
| Video | video_id (PK) | Unique identifier |
| Video | title, description | Metadata |
| Video | uploader_id | Indexed |
| Video | duration, resolution | Encoded ladder info |
| Video | status | uploading, processing, ready, failed |
| Video | storage_path | Object storage key for master |
| Video | view_count | Eventually consistent counter |
| Segment | video_id, bitrate, segment_index | Manifest references |

Storage choice:
- **Video master and segments:** Object storage (S3-like) — cheap, durable, high throughput, designed for large blobs.
- **Metadata:** Relational DB (PostgreSQL) sharded by video_id — needs strong consistency for metadata and status transitions.
- **View counts:** Eventually consistent counter (Redis or Cassandra) incremented asynchronously; reconciled periodically.

## 5. High-level design

```
Uploader -> API Gateway -> Upload Service -> Object Storage (master)
                                           |
                                    Encoding Service
                                           |
                                    Object Storage (segments per bitrate)
                                           |
                                    CDN edge caches
                                           |
                                    Viewer Client (HLS/DASH player)
```

**Upload path:** Client → Upload Service → get presigned URL → client PUTs parts directly to object storage → client calls complete → Encoding Service triggered (event) → transcodes into bitrates → stores segments → metadata marked `ready`.

**Playback path:** Client → GET manifest (from CDN or origin) → client picks bitrate → GET segments from CDN edge → player assembles playback.

**CDN:** Cache popular segments at the edge. A viral video's segments are served from CDN, not origin, drastically reducing origin load and cost.

## 6. Deep dive: encoding pipeline and adaptive bitrate

This is the hard part. A raw upload must become playable on many devices/networks.

| Decision | Options | Tradeoff |
|----------|---------|----------|
| Transcoding | Do it synchronously on upload vs async via job queue | Async is required; synchronous would time out and block the upload |
| Bitrate ladder | Fixed (e.g., 240p, 480p, 720p, 1080p) vs per-title adaptive | Fixed is simpler; per-title saves storage/bandwidth but costs more compute |
| Segment duration | 2–6 seconds (HLS/DASH) | Shorter segments → faster start, more manifest overhead |
| Storage | Store all bitrates in object storage + CDN | Simple, but storage cost is high; use cold tier for old/rare videos |

**Recommended:** Async encoding via a job queue (e.g., SQS → worker fleet). On upload complete, publish an `encoding_job` event. Workers pick up jobs, transcode the master into an adaptive bitrate ladder (HLS/DASH segments), write segments back to object storage, and mark the video `ready`.

**Adaptive bitrate:** The manifest lists variant playlists at different bitrates/resolutions. The player measures its own throughput and switches segments mid-playback. This is a client decision; the server just serves the manifest and segments.

**Cold storage:** Videos older than N days or with zero views in a window move to a cheaper tier (Glacier-class). Rehydrate on demand with a delay, or keep a short preview segment hot.

## 7. Bottlenecks and tradeoffs

- **Viral video / hot segment:** One video watched by millions overloads its origin and one CDN edge. Mitigation: CDN caching (segments are cacheable), shard videos across multiple object-storage buckets/regions, pre-warm CDN for known-big releases.
- **Upload throughput:** Large uploads consume app-server bandwidth if proxied. Use direct-to-object-storage presigned uploads so app servers only handle metadata.
- **Encoding backlog:** Transcoding is compute-heavy. Scale the worker fleet with the queue depth; use spot instances for cost. Prioritize new/popular uploads.
- **Storage cost:** 1.5 PB+ active catalog is expensive. Move cold/long-tail video to cheaper tier; store only popular bitrates hot, transcode-on-demand for rare requests.
- **Manifest staleness:** After encoding completes, the manifest must be visible. Propagate via CDN cache invalidation or short manifest TTL.
- **View count accuracy:** Counting every play synchronously slows playback. Increment asynchronously (queue → batch update) and accept eventual consistency.
- **Year 3 growth:** Petabytes of video; storage cost dominates. Aggressive tiering, per-title encoding ladders, and segment dedup (same video at same bitrate stored once) are required.
- **Monitoring and alerts:** Track video start time (p95/p99), rebuffer ratio, CDN hit rate, origin bandwidth, encoding queue depth, and failed uploads. Page on start time exceeding SLA or rebuffer ratio spiking.

## Follow-up questions to practice

1. How do you support live streaming (low latency, continuous ingest) vs on-demand?
2. How do you protect content so only paying users can watch (DRM, signed URLs)?
3. How do you reduce encoding cost without hurting quality (per-title encoding, spot instances)?
4. What happens when a video segment is missing or corrupt mid-playback?
5. How do you handle upload resumes for large files after a network failure?
