# How to Reclaim Disk Space in MongoDB Atlas

**Article Status:** Technical review, Revision 1.
**Visibility:** Customer.

_[Article Summary]_

As outlined in the [MongoDB Atlas guide on freeing up disk space](https://support.mongodb.com/article/000019603), it's important to understand that the WiredTiger storage engine operates on a no-overwrite principle. Disk space is only released when reusable blocks are identified and utilized for writing new data, before extending the file. Therefore, there's typically no need for manual data compaction since WiredTiger handles this process automatically.

However, if you do find yourself in a situation where you need to reclaim disk space, you may consider initiating an initial sync. Dispensing on various factors such as data size and latency between nodes (particularly in Multi-region clusters), this sync process can span from several hours to even days. To assess whether the available space justifies the effort of performing an initial sync, you can use the following script.

_[Answer]_

The script below will display the quantity of vacant space available for reuse according to WiredTiger, as reflected in the output of the `db.collection.stats()` command, specifically under the **wiredTiger.block-manager.file bytes available for reuse** heading:

## Procedure

1. Connect to your cluster via mongosh.
2. Create variables to hold the total collection size and total reusable size:
```js
var totalSize = 0;
var totalReusableSize = 0;
```
3. Execute the following script:
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
4. Print the output to get the total collection size and total reusable size:
```js
print("\nTotal Storage Size: " + totalSize + " bytes"),("Total Reusable Size: " + totalReusableSize + " bytes");
```