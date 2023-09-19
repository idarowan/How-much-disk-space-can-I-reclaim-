# How to Reclaim Disk Space in MongoDB Atlas

**Article Status:** Technical review, Revision 1.
**Visibility:** Customer.

_[Article Summary]_

This article explains how to determine the amount of disk space that can be reclaimed in MongoDB Atlas. MongoDB Atlas uses the WiredTiger storage engine, which automatically manages disk space by reusing blocks as needed. Manual data compaction is usually unnecessary, but in specific situations, you may want to assess whether reclaiming disk space is warranted. This article provides a script to help you evaluate the available space for reuse.

_[Answer]_

If you’re considering reclaiming disk space in MongoDB Atlas, follow these steps to determine the amount of space available for reuse according to WiredTiger:

## Procedure:

1. Connect to your cluster via mongosh.
2. Create variables to hold the total collection size and total reusable size:
```js
var totalSize = 0;
var totalReusableSize = 0;
```
1. Execute the following script:
```js
db.getMongo().getDBNames().forEach(function (d) {
    if (!["admin", "config", "local"].includes(d)) {
        db.getSiblingDB(d).getCollectionNames().forEach(function (coll) {
            var stat = db.getSiblingDB(d).getCollection(coll).stats();
            var namespace = stat.ns;
            var count = stat.count;
            var storageSize = stat.storageSize;
            var reusableSize = stat.wiredTiger["block-manager"]["file bytes available for reuse"];
            totalSize += storageSize;
            totalReusableSize += reusableSize;
            printjson({
                'namespace': namespace,
                'count': count,
                'storageSize': storageSize,
                'file bytes available for reuse': reusableSize
            });
        });
    }
});
```
1. Print the output to get the total collection size and total reusable size:
```js
print("\nTotal Storage Size: " + totalSize + " bytes"),("Total Reusable Size: " + totalReusableSize + " bytes");
```
This script calculates the total storage size and the total amount of reusable space for all collections in your MongoDB Atlas cluster. By running this script, you can assess whether it’s worthwhile to initiate any actions to reclaim disk space. Remember that WiredTiger handles disk space management automatically, so manual compaction is typically unnecessary unless specific circumstances require it.

For more information on disk space management in MongoDB Atlas, refer to the MongoDB Atlas guide on freeing up disk space.
