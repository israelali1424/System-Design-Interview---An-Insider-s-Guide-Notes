## Key Terms & Definitions

- Autocomplete: A feature that presents a list of suggestions or completions as a user types in a search query.
- Typeahead: Another term for autocomplete, referring to the feature that provides suggestions while a user is typing.
- Search-as-you-type: Similar to autocomplete and typeahead, it describes the functionality of showing search suggestions in real-time.
- Incremental search: The process of displaying search suggestions or results as a user progressively enters their query.

## Clarifying Questions & High-Level Design

The initial step in tackling a system design interview question is to clarify the requirements. Here is an example of a candidate-interviewer interaction:

- Matching: Autocomplete suggestions are supported only at the beginning of a search query.
- Number of Suggestions: The system should return 5 autocomplete suggestions.
- Selection of Suggestions: The system determines the suggestions based on query popularity, determined by historical query frequency.
- Spell Check: Spell check or autocorrect is not supported.
- Language: Search queries are in English.
- Character Constraints: All search queries have lowercase alphabetic characters.
- User Base: The product has 10 million Daily Active Users (DAU).

### Requirements Summary:

- Fast Response Time: Autocomplete suggestions must appear quickly (within 100 milliseconds).
- Relevance: Suggestions should be relevant to the search term.
- Sorting: Results must be sorted by popularity or other ranking models.
- Scalability: The system should handle high traffic volume.
- High Availability: The system remains available and accessible even during partial outages, slowdowns, or network errors.

## Deep Dive into Design Aspects

### Back of the Envelope Estimation:

- 10 million DAU, with an average of 10 searches per user per day.
- 20 bytes of data per query string, assuming ASCII character encoding.
- Each query contains 4 words, with an average of 5 characters per word.
- On average, 20 requests are sent to the backend for each search query.
- ~24,000 queries per second (QPS).
- Peak QPS: Double the average QPS, around 48,000.
- Approximately 0.4 GB of new data is added to storage daily.

### High-Level Design:

The system is divided into two main components:

1. Data Gathering Service: Gathers user input queries and aggregates them in real-time.
2. Query Service: Returns the top 5 most frequently searched terms given a search query or prefix.

#### Data Gathering Service:

- Collects user input queries and updates a frequency table in real-time.
- Frequency table stores the query string and its frequency.

#### Query Service:

- Given a search query or prefix, returns the 5 most frequently searched terms.
- Utilizes a frequency table to retrieve the relevant suggestions.
- Executes a SQL query to fetch the top 5 frequently searched queries.

## Comparison of Different Techniques

In this context, the primary technique discussed is using a frequency table to store and retrieve autocomplete suggestions. However, as the data set grows, accessing the database directly becomes a bottleneck. Further optimization is required, which will be explored in subsequent sections.

## Step 3 - Design Deep Dive

In this section, we will dive deep into a few components of the search autocomplete system and explore optimizations. The components we will focus on are:

- Trie Data Structure
- Data Gathering Service
- Query Service
- Scaling the Storage
- Trie Operations

### Trie Data Structure

To overcome the inefficiency of fetching top 5 search queries from a relational database, we can utilize the trie (prefix tree) data structure. Trie allows for more efficient retrieval of autocomplete suggestions. Here's an overview of the trie data structure:

- Trie is a tree-like data structure designed for string retrieval operations.
- Each node represents a character and has 26 children, one for each possible character (assuming lowercase alphabetic characters).
- Each node represents a word or a prefix string.
- To support sorting by frequency, frequency information needs to be included in nodes.

By incorporating the trie data structure into the system, we can optimize the response time of autocomplete queries.

### Trie Operations

To retrieve the top k most searched queries efficiently, we can follow these steps:

1. Find the prefix: Locate the node corresponding to the given prefix. Time complexity: O(1) by limiting the maximum length of a prefix.
2. Traverse the subtree: Traverse the subtree from the prefix node to get all valid children that can form valid query strings. Time complexity: O(c), where c is the number of children.
3. Sort and get top k: Sort the children based on frequency and retrieve the top k queries. Time complexity: O(clogc).

These operations ensure that the autocomplete suggestions are relevant and sorted by popularity.

### Optimization: Limit the Max Length of a Prefix

Since users rarely type long search queries, we can limit the length of a prefix to a small integer, such as 50. This reduces the time complexity of finding the prefix from O(p) to O(1).

### Optimization: Cache Top Search Queries at Each Node

To avoid traversing the entire trie for every autocomplete query, we can cache the top k most frequently searched queries at each node. This optimization significantly reduces the time complexity of retrieving the top queries. However, it requires additional space to store the cached queries at every node. Trading space for time is a worthwhile tradeoff for faster response times.

### Data Gathering Service

Real-time updating of the trie for every search query is not practical due to the high query volume and limited changes to the top suggestions. To design a scalable data gathering service, we need to consider where the data comes from and how it is used. In most cases, the data comes from analytics or logging services.

Components of the data gathering service include:

- Analytics Logs: Stores raw data about search queries in an append-only, non-indexed format.
- Aggregators: Aggregate the raw data from the analytics logs to prepare it for processing.
- Aggregated Data: Stores the aggregated data, which is used to rebuild the trie.

The frequency of rebuilding the trie depends on the use case. Real-time applications like Twitter may require more frequent rebuilding, while other applications may rebuild the trie less frequently, such as once per week.

### Scaling the Storage

As the system scales and handles a larger user base, storage scalability becomes a concern. One approach to handle the growing storage requirements is to use distributed file systems or cloud storage solutions. These solutions provide horizontal scalability, allowing the system to handle increasing amounts of data efficiently.

## Comparison of Different Techniques

The main technique discussed in this section is the use of the trie data structure for efficient autocomplete suggestions. By utilizing trie operations and optimizing the trie design, we can achieve fast response times and relevant suggestions. The optimizations include limiting the length of a prefix and caching top search queries at each node.

Other techniques, such as using relational databases for storage or alternative data structures, may not offer the same level of efficiency for autocomplete systems. The trie data structure proves to be a suitable choice due to its ability to compactly store strings and support fast retrieval operations.

## Step 4 - Wrap Up

After diving deep into the design of the search autocomplete system, let's address some follow-up questions and wrap up the discussion.

### Supporting Multiple Languages

To support non-English queries, Unicode characters are stored in trie nodes. Unicode is an encoding standard that covers characters for all writing systems worldwide. By incorporating Unicode support, the system can handle queries in various languages.

### Different Top Search Queries in Different Countries

If the top search queries vary among different countries, one approach is to build different tries for each country. This allows for personalized autocomplete suggestions based on regional preferences. To improve response time, the tries can be stored in Content Delivery Networks (CDNs) strategically placed closer to the users.

### Supporting Trending (Real-time) Search Queries

The original design of the system may not be suitable for supporting trending or real-time search queries. However, a few ideas can be explored to address this:

- Sharding: Reduce the working data set by sharding the data and processing it in parallel.
- Adjust Ranking Model: Assign more weight to recent search queries to give them higher priority in autocomplete suggestions.
- Stream Processing: If data is generated continuously in streams, a different set of systems such as Apache Hadoop MapReduce, Apache Spark Streaming, Apache Storm, or Apache Kafka can be employed for stream processing. These technologies specialize in handling streaming data and enable real-time processing of the search queries.

Keep in mind that implementing a real-time search autocomplete system is a complex task that requires specific domain knowledge and consideration of various factors.

Please note that visual aids are not provided in this excerpt.



