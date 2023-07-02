# Chapter 9: Design a Web Crawler

## Introduction

- Web crawlers, or spiders, are tools used by search engines to discover and update web content.
- The complexity of a web crawler depends on the scale and the required features.
- The basic operation of a crawler involves downloading web pages, extracting new URLs from these pages, and adding them to the list of URLs to be downloaded.

## Interview Requirements

- During a design interview, you may need to understand the following requirements:
  - The purpose of the crawler (e.g., for search engine indexing)
  - The number of web pages to be collected
  - The types of content to be included
  - Consideration for newly added or edited pages
  - How long the crawled pages will be stored
  - How duplicate content will be handled

## Characteristics of a Good Web Crawler

- A good web crawler should be scalable, robust, polite, and extensible.
- To download 1 billion pages per month, a web crawler needs to handle approximately 400 pages per second and have storage for around 500 TB per month.

## High-Level Design

- The high-level design includes components like seed URLs, URL Frontier, HTML Downloader, DNS Resolver, Content Parser, Content Seen?, Content Storage, URL Extractor, URL Filter, URL Seen?, and URL Storage.
- The workflow involves:
  - Adding seed URLs to the URL Frontier
  - Downloading and parsing HTML pages
  - Checking for duplicated content
  - Extracting links from HTML pages
  - Filtering links
  - Checking for already visited URLs
  - Adding new URLs to the URL Frontier

## In-Depth Design Considerations

- Depth-First Search vs. Breadth-First Search: BFS is commonly used by web crawlers as DFS can result in very deep traversals, potentially leading to infinite loops in the face of circular references.
- Implementing the URL Frontier: This is a data structure to store the URLs to be downloaded, ensuring politeness (not overwhelming servers with requests), and managing priorities and content freshness.
- HTML Downloader: This component is responsible for downloading web pages and needs to respect the Robots Exclusion Protocol.
- Robustness: To ensure robustness, the system must handle failures gracefully, including load distribution, frequent saving of states and data, exception handling, and data validation.
- Extensibility: The system should be designed to easily plug in new modules, allowing it to adapt to future requirements and technological advancements.
- Avoiding Problematic Content: Filters can be used to avoid downloading inappropriate or illegal content, based on the crawler's purpose.

# Web Crawling Design

## BFS and URL Prioritization

- A standard Breadth-First Search (BFS) does not prioritize URLs based on factors such as PageRank, web traffic, or update frequency.
- A data structure like URL frontier is required to address this issue and enhance the BFS approach.

## URL Frontier

- URL frontier is essentially a data structure that stores URLs to be downloaded.
- This structure is critical to maintain politeness, prioritize URLs, and ensure the freshness of the content.

### Politeness

- Politeness in web crawling implies avoiding sending excessive requests to the same host server in a short span of time. Overloading a server with too many requests is considered impolite or even a denial-of-service (DOS) attack.
- Politeness is generally implemented by enforcing a download delay between two tasks from the same host and limiting the process to one page at a time.
- The design that helps in managing politeness includes a queue router, a mapping table, FIFO queues, a queue selector, and worker threads.
  - The queue router ensures each queue contains URLs from the same host.
  - The mapping table assigns each host to a specific queue.
  - FIFO queues store URLs from the same host.
  - The queue selector maps each worker thread to a FIFO queue and controls the URL selection process.
  - The worker threads download web pages one-by-one from the same host with a delay between tasks.

### Priority

- URLs are prioritized based on their usefulness, which is determined by measures like PageRank, website traffic, and update frequency.
- A component known as "Prioritizer" handles the URL prioritization.
- The design managing URL priority includes:
  - The prioritizer, which accepts URLs as input and computes the priorities.
  - Multiple queues, each of which has an assigned priority. High-priority queues are more likely to be selected.
  - A queue selector, which randomly chooses a queue, favoring those with higher priority.

### Freshness

- Web pages constantly change—new content is added, old content is deleted, and existing content is edited. To keep the crawled data fresh, the web crawler must periodically recrawl the downloaded pages.
- This process can be optimized by:
  - Recrawling based on the web pages' update history.
  - Prioritizing URLs to recrawl important pages first and more frequently.

## Storage for URL Frontier

- In real-world scenarios, the number of URLs in the frontier could be in the hundreds of millions, making in-memory storage infeasible and unscalable.
- The system utilizes a hybrid storage approach: The majority of URLs are stored on the disk while buffers in memory handle enqueue/dequeue operations. Data in the buffer is written to the disk periodically.

## HTML Downloader

- The HTML Downloader uses the HTTP protocol to download web pages from the internet.
- It must respect the Robots Exclusion Protocol by checking the robots.txt file of a website before attempting to crawl it.

### Robots.txt

- Robots.txt, or the Robots Exclusion Protocol, communicates which pages crawlers are allowed or not allowed to download from a website.
- To optimize the process, results of the robots.txt file are cached to avoid repeat downloads.

### Performance Optimization for HTML Downloader

- Distributed crawl: By distributing crawl jobs across multiple servers (each running multiple threads), the system achieves high performance.
- Cache DNS Resolver: The system maintains its DNS cache to prevent frequent DNS calls, enhancing speed.
- Locality: Distributing crawl servers geographically ensures faster download times due to reduced network latency.
- Short timeout: The system avoids prolonged waits by
