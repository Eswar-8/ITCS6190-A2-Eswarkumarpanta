# Assignment #2 — Report

**Name:** Eswar kumar

**Student ID:** 801505752

**Email:** epanta@charlotte.edu

---

## Design

Which design did you choose (A, B, or your own)? Explain in your own words:

I chose Design A 
- What your **Mapper** emits as key and value, and why that is the right thing to emit.
`My mapper reads each input line and extracts two things: the document ID (the first word) and the document text (everything after). It then tokenizes the text by converting to lowercase, splitting on whitespace, and removing any non-alphanumeric characters. Since we only care about unique words per document, I store the tokens in a HashSet to remove duplicates. The mapper emits the document ID as the key and a space-separated list of unique words as the value. This is the right approach because it gives the reducer all the information needed about each document in a compact form—just the words that matter, without repetition.`
- What your **Reducer** receives for one key, what it does with it, and where the Jaccard
  similarity is computed.
  `The reducer receives one key-value pair per document. The key is the document ID, and the value is the word list. I store each document's word set in a HashMap so I can access them later. I don't do any comparison in the reduce() method itself—I just accumulate the documents.
Once all documents have been received, the cleanup() method runs. This is where I compare every unique pair of documents. For each pair, I calculate the intersection (shared words) and union (all unique words) using Java's Set operations, then compute the Jaccard similarity as the ratio of intersection size to union size.`
- What you had to set in the **Driver** beyond what L4's `Controller` set, and why.
`First, I set the number of reducers to exactly 1 with job.setNumReduceTasks(1). This is essential for Design A—if I used multiple reducers, documents would be split across them and I'd never see all pairs in one place to compare.
Second, I set the output types explicitly: mapper output as (Text, Text) and final output as (Text, NullWritable). The final value is NullWritable because the entire output line (the similarity pair) is the key—there's no separate value to emit.
Third, I needed to import my Mapper and Reducer classes and wire them to the job, which the skeleton doesn't do automatically. The driver is what brings all three pieces together.`


---

## How I ran it

The commands you used, in the order you used them. If you deviated from the steps in the
README, say where and why.

```bash
docker compose up -d

# Waited ~60 seconds, then verified http://localhost:9870 showed 3 DataNodes

# Step 3: Build the project
mvn clean package

# Step 4: Copy JAR and datasets to container
docker cp target/DocumentSimilarity-0.0.1-SNAPSHOT.jar resourcemanager:/tmp/
docker cp shared-folder/input/data/small_dataset.txt resourcemanager:/tmp/
docker cp shared-folder/input/data/dataset.txt resourcemanager:/tmp/

# Step 5: Open shell in container
docker exec -it resourcemanager bash
cd /tmp

# Step 6: Load datasets into HDFS
hadoop fs -mkdir -p /input/data
hadoop fs -put ./small_dataset.txt /input/data
hadoop fs -put ./dataset.txt /input/data
hadoop fs -ls /input/data

# Step 7: Run job on small dataset
hadoop jar /tmp/DocumentSimilarity-0.0.1-SNAPSHOT.jar \
  com.example.controller.DocumentSimilarityDriver /input/data/small_dataset.txt /output/small_dataset

hadoop fs -cat /output/small_dataset/*

# Step 8: Run job on full dataset
hadoop jar /tmp/DocumentSimilarity-0.0.1-SNAPSHOT.jar \
  com.example.controller.DocumentSimilarityDriver /input/data/dataset.txt /output/dataset

hadoop fs -cat /output/dataset/*

# Step 9: Copy results back
hdfs dfs -get /output /tmp/
exit

# Back on local machine:
docker cp resourcemanager:/tmp/output/. shared-folder/output/

# Step 10: Stop the cluster
docker compose down
```

---

## Output

### `small_dataset.txt` (3 lines)

```
Document1, Document2 Similarity: 0.18
Document1, Document3 Similarity: 0.20
Document2, Document3 Similarity: 0.10
```

### `dataset.txt` (66 lines)

```
Doc01, Doc02 Similarity: 0.16
Doc01, Doc03 Similarity: 0.13
Doc01, Doc04 Similarity: 0.07
Doc01, Doc05 Similarity: 0.10
Doc01, Doc06 Similarity: 0.09
Doc01, Doc07 Similarity: 0.11
Doc01, Doc08 Similarity: 0.10
Doc01, Doc09 Similarity: 0.11
Doc01, Doc10 Similarity: 0.09
Doc01, Doc11 Similarity: 0.07
Doc01, Doc12 Similarity: 0.19
Doc02, Doc03 Similarity: 0.20
Doc02, Doc04 Similarity: 0.13
Doc02, Doc05 Similarity: 0.10
Doc02, Doc06 Similarity: 0.09
Doc02, Doc07 Similarity: 0.06
Doc02, Doc08 Similarity: 0.09
Doc02, Doc09 Similarity: 0.05
Doc02, Doc10 Similarity: 0.10
Doc02, Doc11 Similarity: 0.06
Doc02, Doc12 Similarity: 0.14
Doc03, Doc04 Similarity: 0.17
Doc03, Doc05 Similarity: 0.11
Doc03, Doc06 Similarity: 0.08
Doc03, Doc07 Similarity: 0.16
Doc03, Doc08 Similarity: 0.11
Doc03, Doc09 Similarity: 0.07
Doc03, Doc10 Similarity: 0.10
Doc03, Doc11 Similarity: 0.12
Doc03, Doc12 Similarity: 0.11
Doc04, Doc05 Similarity: 0.09
Doc04, Doc06 Similarity: 0.11
Doc04, Doc07 Similarity: 0.18
Doc04, Doc08 Similarity: 0.09
Doc04, Doc09 Similarity: 0.08
Doc04, Doc10 Similarity: 0.10
Doc04, Doc11 Similarity: 0.09
Doc04, Doc12 Similarity: 0.09
Doc05, Doc06 Similarity: 0.20
Doc05, Doc07 Similarity: 0.14
Doc05, Doc08 Similarity: 0.15
Doc05, Doc09 Similarity: 0.07
Doc05, Doc10 Similarity: 0.13
Doc05, Doc11 Similarity: 0.14
Doc05, Doc12 Similarity: 0.11
Doc06, Doc07 Similarity: 0.17
Doc06, Doc08 Similarity: 0.15
Doc06, Doc09 Similarity: 0.08
Doc06, Doc10 Similarity: 0.10
Doc06, Doc11 Similarity: 0.12
Doc06, Doc12 Similarity: 0.13
Doc07, Doc08 Similarity: 0.15
Doc07, Doc09 Similarity: 0.07
Doc07, Doc10 Similarity: 0.08
Doc07, Doc11 Similarity: 0.12
Doc07, Doc12 Similarity: 0.11
Doc08, Doc09 Similarity: 0.19
Doc08, Doc10 Similarity: 0.13
Doc08, Doc11 Similarity: 0.22
Doc08, Doc12 Similarity: 0.12
Doc09, Doc10 Similarity: 0.13
Doc09, Doc11 Similarity: 0.12
Doc09, Doc12 Similarity: 0.13
Doc10, Doc11 Similarity: 0.12
Doc10, Doc12 Similarity: 0.12
Doc11, Doc12 Similarity: 0.11
```

---

## Analysis

Look at the results for `dataset.txt`.

**Most and least similar pairs:**

The most similar pair is Doc08 and Doc11 with a similarity of 0.22. Other highly similar pairs include Doc05 and Doc06 (0.20), and Doc01 and Doc12 (0.19). The least similar pairs are scattered throughout, with many at 0.05–0.07, such as Doc02 and Doc09 (0.05), Doc01 and Doc04 (0.07), and Doc01 and Doc11 (0.07).

**Do the similarities make sense?**

Without seeing the actual document contents, it's difficult to say definitively whether the high similarities reflect genuine topical overlap. However, if the documents cover a broad range of topics (news articles, research papers, etc.), a maximum similarity of 0.22 suggests they don't share many words. This would make sense if the documents use different vocabulary to discuss different subjects.

**Why are values so low and close together?**

The Jaccard similarity values are consistently low (0.05–0.22) and tightly clustered because most pairs of documents are lexically quite different. The Jaccard similarity is strict: it only counts exact word matches and penalizes every word that appears in one document but not the other. With a diverse document collection and varied vocabulary, most pairs end up sharing only a small fraction of their unique words.

**One change to make numbers more meaningful:**

The most impactful change would be to **remove common stop words** (a, the, is, and, or, etc.) from the tokenization. Stop words appear in nearly every document, so removing them would eliminate noise and make the non-trivial shared vocabulary stand out. This would increase the contrast between unrelated and related documents, making the similarity scores more informative. For example, if Doc08 and Doc11 both discuss technology and currently share 22% of their words, removing stop words might reveal they actually share 40–50% of their meaningful vocabulary.



---

## Scalability

**If you used Design A:** it relies on a single reducer that holds every document in memory.
What concretely breaks when the collection has a million documents? Sketch how Design B
avoids the problem.

```
The single reducer stores all documents' word sets in a HashMap. With a million documents, even if each document's word set averages 100 unique words, that's roughly 100 million word objects in memory. On top of that, the cleanup() method constructs an additional HashSet for each pair comparison—for a million documents, that's roughly 500 billion pairs to compare (n*(n-1)/2). The reducer would:
1. Run out of heap memory trying to store all documents
2. Hit timeout limits during the cleanup() phase because comparing 500 billion pairs takes hours
3. Fail completely if any single document contains millions of words
Concretely, with a typical Hadoop reducer heap size of 2GB, you'd hit an OutOfMemoryError somewhere around 10,000–50,000 documents, depending on vocabulary size.
```
**If you used Design B:** why did it need more than one pass (or how did you avoid that)?
What is its own bottleneck?
```
- For each document ID it sees, it emits (docA, docB) for all previously seen documents
- The reducer receives all info about a specific pair and computes just that pair's similarity

By distributing pairs across many reducers:
- No single reducer holds all documents in memory
- Each reducer processes only a handful of pairs
- The total work is divided by the number of reducers

The trade-off is complexity: the mapper must maintain a sliding window or coordinate with other mappers to avoid duplicates, and you need a way to emit all pairs without exceeding the mapper's memory.
```



---

## Problems and fixes

Anything that went wrong and what resolved it. Paste the actual error message. If nothing
went wrong, say so.
```
There were no Java compilation errors, HDFS errors, or Hadoop runtime exceptions. The only
issue was incomplete code in the skeleton file, which was quickly resolved by implementing
the required logic.
```


---

## Use of generative AI

If you used a generative AI tool, include the acknowledgment statement from the syllabus and
say specifically what you used it for. If you did not use one, say so.
```
I used Claude (an AI assistant made by Anthropic) to help write and debug this MapReduce
job. Specifically, I used it for:

1. **Writing the Mapper class** – implementing the tokenization rules and word set emission
2. **Writing the Reducer class** – implementing the reduce() method to store documents and
   the cleanup() method to compute Jaccard similarity
3. **Writing the Driver class** – configuring the MapReduce job, setting the number of
   reducers to 1, and wiring the Mapper and Reducer
4. **Debugging output issues** – when the job produced no output, I used Claude to help
   identify that the Reducer methods were empty and needed implementation
5. **Understanding Design A vs B** – Claude explained the scalability trade-offs between
   the two designs

I read and understood all generated code before using it, and I verified that the output
matched the expected results.

**Acknowledgment statement:** "I have used Claude, a generative AI tool, to assist in
writing and debugging this assignment. I have reviewed all code generated by the AI and
take responsibility for its correctness and compliance with the assignment requirements."
```


