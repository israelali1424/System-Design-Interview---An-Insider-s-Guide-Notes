# Chapter 15: System Design Interview - Google Drive Design

## Key Terms & Definitions

- **Cloud Storage Services:** These are internet-based services that allow you to save data to a remote database. Examples include Google Drive, Dropbox, Microsoft OneDrive, and Apple iCloud.
- **File Synchronization:** This refers to the process of ensuring that the same files are available on different devices.
- **File Revisions:** These are different versions of a file that have been saved over time.
- **Sharding:** This is a method of splitting and storing a single logical dataset in multiple databases.
- **Metadata:** This is data about data. In this context, it includes data about the files such as user data, login info, file info, etc.
- **Block Servers:** These are servers that break up files into smaller blocks, each with a unique hash value, for storage.
- **Delta Sync:** This is a method of syncing where only the changed parts of a file are synced instead of the whole file.
- **Amazon S3:** This is a scalable object storage service designed for storing and retrieving any amount of data from anywhere.
- **Load Balancer:** This is a device that distributes network traffic across multiple servers to ensure no single server bears too much demand.
- **API servers:** These are responsible for user authentication, managing user profile, updating file metadata, etc.
- **Notification Service:** This is a service that sends alerts when a file is edited, deleted, or shared with you.

## Clarifying Questions & High-Level Design

The chapter begins by clarifying the scope of the project. The important features of the system include file upload and download, file synchronization, and notifications. The system will support both mobile and web applications, and any file type can be uploaded. Files will need to be encrypted and can be up to 10 GB in size.

The proposed high-level design of the system begins with a single server setup, including a web server for file upload and download, a database for metadata, and a storage system for the files. As the system scales, data is sharded across multiple storage servers. The system later transitions to using Amazon S3 for storage.

## Deep Dive into Design Aspects

For large file uploads, a resumable upload strategy is proposed, consisting of three steps:

- Initial request to retrieve the resumable URL
- Uploading the data and monitoring the upload state
- If the upload is interrupted, resume the upload.

The system moves from a single server setup to multiple servers, introducing components such as load balancers, additional web servers, a separate metadata database, and Amazon S3 for file storage.

For handling sync conflicts, the system proposes a strategy where the first version that gets processed wins, and the version that gets processed later receives a conflict. The system presents both copies of the same file to the user to resolve the conflict.

Block servers are used to optimize bandwidth usage by only uploading changed blocks of files (delta sync) and compressing blocks before upload.

## Comparison of Different Techniques

Delta sync is compared to a full file sync. In delta sync, only modified blocks are synced instead of the whole file, which can save significant bandwidth.

In terms of file storage, the chapter compares a single server setup to multiple servers and Amazon S3 storage. While a single server is easier to set up, multiple servers and Amazon S3 provide scalability, availability, and data redundancy.

In resolving sync conflicts, the first version processed wins. This technique is compared to other potential conflict resolution strategies, although specifics are not given.

# Chapter 12: System Design Interview - Notes

## File Operations

- Files are split into blocks.
- Blocks are compressed and encrypted for security.
- Blocks are uploaded to cloud storage.
- Delta sync is used to only transfer modified blocks, saving network traffic.
- Block servers facilitate delta sync and compression.

## Strong Consistency Requirement

- The system requires strong consistency. Metadata cache and database layers should not show different file states to different clients.
- Caches should be invalidated on database write to ensure the cache and the database hold the same value.
- Achieving strong consistency in relational databases is easier due to ACID (Atomicity, Consistency, Isolation, Durability) properties.
- NoSQL databases do not support ACID properties by default; these properties must be incorporated programmatically.
- The design opts for relational databases for native ACID support.

## Metadata Database Schema

- Contains tables for User, Device, Namespace, File, File_version, and Block.
- Namespace refers to the root directory of a user.
- File_version table stores version history of a file, with existing rows being read-only for data integrity.
- Block table stores data related to file blocks, with files being reconstructable from their constituent blocks.

## Upload Flow

- The upload process involves two parallel requests: add file metadata and upload the file to cloud storage.
- File metadata addition involves changing the file upload status to “pending,” notifying the notification service of a new file being added, and informing relevant clients.
- File upload involves breaking the file into blocks, compressing and encrypting the blocks, and uploading them to cloud storage.
- The file status is updated to “uploaded” upon successful upload, and relevant clients are notified.

## Download Flow

- Triggers when a file is added or edited elsewhere.
- A client learns of a file change via the notification service or by pulling the latest data when coming back online.
- Upon learning of a change, a client requests metadata via API servers and downloads blocks to construct the file.

## Notification Service

- Ensures file consistency by informing other clients of local mutations.
- Communication options include Long polling and WebSocket.
- Long polling is chosen due to its unidirectional nature and sufficiency for infrequent notifications.
- Clients maintain a long poll connection to the notification service and connect to the metadata server to download the latest changes upon connection closure or timeout.

## Saving Storage Space

- Techniques for storage reduction include data block de-duplication, intelligent backup strategies, and moving infrequently used data to cold storage.
- Backup strategies could involve setting a limit to the number of stored versions or only retaining valuable versions.

## Failure Handling

- Failures can occur in load balancers, block servers, cloud storage, API servers, metadata cache servers, metadata DB, notification service, and offline backup queue.
- Different strategies are used to handle failures, such as redundancy, fallbacks, data replication, and server swapping.

## System Design Trade-offs and Choices

- Directly uploading files to cloud storage from the client can make uploads faster but could be error-prone, and pose security issues.
- Shifting online/offline logic to a separate "presence service" can facilitate integration with other services.

## Reference

[1] Google Drive: https://www.google.com/drive/
[2] Upload file data: https://developers.google.com/drive/api/v2/manage-uploads
[3] Amazon S3: https://aws.amazon.com/s3
[4] Differential Synchronization https://neil.fraser.name/writing/sync/
[5] Differential Synchronization youtube talk https://www.youtube.com/watch?v=S2Hp_1jqpY8
[6] How We’ve Scaled Dropbox: https://youtu.be/PE4gwstWhmc
[7] Tridgell, A., & Mackerras, P. (1996). The rsync algorithm.
[8] Librsync. (n.d.). Retrieved April 18, 2015, from https://github.com/librsync/librsync
[9] ACID: https://en.wikipedia.org/wiki/ACID
[10] Dropbox security white paper: https://www.dropbox.com/static/business/resources/Security_Whitepaper.pdf
[11] Amazon S3 Glacier: https://aws.amazon.com/glacier/faqs/
