# Design Facebook status system

Design a system where a user can post a status message and other users can search text across all the status messages

## High-level design

![fb-status](https://user-images.githubusercontent.com/7844980/147387254-d8aa82fb-fd00-4ad1-8ba9-785931560858.png)

#### Create a post
1. All post created will be stored directly in the database
    * `Create Post Service` will generate a unique id for each post
    * Schema for a post is: `<post_id, post_content, timestamp>`
2. Database is set up with CDC and dumps all the new posts into a queue
3. A batch process will read from the queue every 5 minutes and generate indexed data
4. The indexed data will be dump to in-memory index and snapshot files
    * Snapshot files are AppendLog files to rebuild the index in the future

## Batch process
* Batch process is a MapReduce job that takes in records of newly created posts
* The Mapper will parse each post and for each keyword, it'll map it to a post id
* The Reducer will go through each key and combine all post id that belongs to the same keyword
* To rebuild index from snapshot files, we just have to re-run the Reducer against the snapshot files

![fb-status-mapreducer](https://user-images.githubusercontent.com/7844980/147387975-7b4113c9-c552-4762-b234-2d165345600d.png)
