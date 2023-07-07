# Key Terms & Definitions

- **YouTube**: A platform where content creators upload videos and viewers can play them. It supports various features like commenting, sharing, liking a video, saving a video to playlists, and subscribing to a channel.
- **Video Streaming**: The continuous transmission of video files from a server to a client.
- **CDN (Content Delivery Network)**: A system of distributed servers that deliver content to a user, based on the geographic locations of the user.
- **Blob Storage**: A service for storing large amounts of unstructured object data, such as text or binary data.
- **Transcoding**: The process of converting a video file from one format to another.
- **Metadata**: Data that provides information about other data. In this context, it refers to information about a video, such as its title, description, tags, and other related data.
- **Video Resolution**: The number of distinct pixels that can be displayed in each dimension of a video. It is usually quoted as width × height.
- **Streaming Protocols**: Standardized methods to control data transfer for video streaming. 

# Clarifying Questions & High-Level Design

In this design interview scenario, the interviewer asked the candidate to design a system similar to YouTube. The key features of this system include the ability to upload a video and watch a video, with a specific focus on supporting mobile apps, web browsers, and smart TVs. The system needs to support a large number of international users and offer various video resolutions. The system should also feature robust security measures, including encryption, and have a file size limit for videos of 1GB.

The high-level design of this system is divided into three major components:

- **Clients**: These include devices like computers, mobile phones, and smart TVs that users employ to access the system.
- **CDN**: Videos are stored in the CDN and streamed from it when a user presses play.
- **API servers**: These handle all other operations except video streaming, such as feed recommendation, generating video upload URLs, updating metadata databases and caches, and user signups.

# Deep Dive into Design Aspects

The system design is further refined with a focus on two important flows: video uploading and video streaming.

- **Video Uploading**: This flow involves users uploading videos to original storage, from where they are fetched by transcoding servers and transcoded into multiple formats. Once this process is complete, the transcoded videos are sent to the transcoded storage and distributed to the CDN, while completion events are queued in a completion queue. Completion handler workers then pull event data from the queue and update the metadata database and cache.
- **Video Streaming**: This flow involves streaming the video from the CDN directly to the client. The edge server closest to the user delivers the video, ensuring low latency.

# Comparison of Different Techniques

Different streaming protocols are used for controlling data transfer for video streaming. Some popular protocols include MPEG-DASH, Apple HLS, Microsoft Smooth Streaming, and Adobe HTTP Dynamic Streaming (HDS). Choosing the right streaming protocol for the system is crucial based on the use cases.

Video transcoding is a critical aspect of the system design, and the specific requirements of content creators can vary widely. Some might require watermarks on their videos, some might provide their own thumbnail images, while others may upload videos of different definitions. To accommodate these various requirements and maintain high parallelism, the design adopts a directed acyclic graph (DAG) model, allowing for the flexible definition and execution of tasks.


# System Design Interview - Chapter 12: Video Transcoding Architecture

_by Alex Xu_

## Video Transcoding Architecture Components

1. **Preprocessor:** Handles video splitting, DAG generation, and caching data for better reliability. In video splitting, the stream is divided into smaller Group of Pictures (GOP), which are independent, playable units.
2. **DAG Scheduler:** Splits a DAG graph into stages of tasks and schedules them in the resource manager's task queue.
3. **Resource Manager:** Manages resource allocation efficiency and contains a task queue, worker queue, and running queue.
4. **Task Workers:** Execute the tasks defined in the DAG.
5. **Temporary Storage:** Multiple storage systems are used based on factors like data type, size, access frequency, and lifespan.
6. **Encoded Video:** The final output of the encoding pipeline. For instance, `funny_720p.mp4`.

## System Optimizations

- **Speed optimization:** Includes techniques such as parallelizing video uploading, placing upload centers close to users, and achieving high parallelism.
- **Safety optimization:** Includes the use of pre-signed upload URLs for ensuring that only authorized users upload videos, and different methods to protect copyrighted videos like Digital rights management (DRM) systems, AES encryption, and visual watermarking.
- **Cost-saving optimization:** Optimizations include serving popular videos from CDN and other videos from high capacity storage video servers, encoding short videos on-demand, regional distribution, and building your own CDN.

## Error Handling

- **Recoverable error:** The general idea is to retry the operation a few times, and if the task continues to fail, return a proper error code to the client.
- **Non-recoverable error:** The system stops the running tasks associated with the video and returns the proper error code to the client.

## Additional Considerations

- Scaling the API tier and the database.
- Live streaming considerations, including the higher latency requirement and different sets of error handling.
- Video takedowns due to violations of copyrights or illegal acts.

## Conclusion

Understanding this architecture design is essential for dealing with large-scale video streaming services.

## References

1. [YouTube by the numbers](https://www.omnicoreagency.com/youtube-statistics/)
2. [2019 YouTube Demographics](https://blog.hubspot.com/marketing/youtube-demographics)
3. [Cloudfront Pricing](https://aws.amazon.com/cloudfront/pricing/)
4. [Netflix on AWS](https://aws.amazon.com/solutions/case-studies/netflix/)
5. [Akamai homepage](https://www.akamai.com/)
6. [Binary large object](https://en.wikipedia.org/wiki/Binary_large_object)
7. [Here’s What You Need to Know About Streaming Protocols](https://www.dacast.com/blog/streaming-protocols/)
8. [SVE: Distributed Video Processing at Facebook Scale](https://www.cs.princeton.edu/~wlloyd/papers/sve-sosp17.pdf)
9. [Weibo video processing architecture](https://www.upyun.com/opentalk/399.html)
10. [Delegate access with a shared access signature](https://docs.microsoft.com/en-us/rest/api/storageservices/delegate-access-with-shared-access-signature)
11. [YouTube scalability talk by early YouTube employee](https://www.youtube.com/watch?v=w5WVu624fY8)
12. [Understanding the characteristics of internet short video sharing: A youtube-based measurement study](https://arxiv.org/pdf/0707.3670.pdf)
13. [Content Popularity for Open Connect](https://netflixtechblog.com/content-popularity-for-open-connect-b86d56f613b)
