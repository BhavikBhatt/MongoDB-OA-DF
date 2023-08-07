# MongoDB-OA-DF
MongoDB Atlas - Online Archive and Data Federation Lab

## Prerequisites 
1. Login to MongoDB Atlas and spin up a dedicated cluster (M10+)
2. Make sure at least 1 user with read-write access has been added.
3. Make sure that IP Access List allows access (add Allow ACCESS FROM ANYWHERE(0.0.0.0\0) to be sure).
4. Type the following command into your terminal, to get the sample dataset:
   
   ```
   git clone git@github.com:BhavikBhatt/MongoDB-OA-DF.git
   ```

# Online Archive Lab

## Steps
### 1. Ingest sample data into your MongoDB Atlas cluster

Go to the ```Connect``` button of the Atlas cluster, and copy the MongoDB Drivers connection string (paste this in a notepad).

Navigate to the git directory that was cloned via the terminal/command line.

In your terminal, paste the following, and replace the URI with your MongoDB connection string (replace placeholder credentials with your username/password):
```
mongoimport --uri="<connection string>" --db=OA-LAB --collection=sales --file sales.json
```
Navigate to your cluster's collections to explore the sales collection, which should have 5,000 documents

### 2. Run Sample Queries on the ```OA-LAB.sales``` Collection

a. Run a query to get the sales with ```sale_date``` before January 1st, 2015. Do this in the aggregation builder (text builder).
```
[
  {
    $match:
      {
        saleDate: {
          $lt: ISODate("2015-01-01"),
        },
      },
  },
  {
    $count:
      "count",
  },
]
```

b. Run a query to get the number of sales grouped by ```storeLocation```. 
```
[
  {
    $group:
      {
        _id: "$storeLocation",
        count: {
          $sum: 1,
        },
      },
  },
  {
    $sort:
      {
        count: -1,
      },
  },
]
```

### 3. Create Indexes on the ```OA-LAB.sales``` collection

Create one index that is:
```
{
  "saleDate": 1
}
```

And another index that is:
```
{
  "storeLocation": 1
}
```

### 4. Configure Date Match Online Archive
 
Navigate to the 'Online Archive' tab of your cluster -> Configure Online Archive -> Next.

Set the following:

a. Namespace: ```OA-LAB.sales```

b. DateField: ```saleDate```

c. Age Limit: ```3,200``` (this will effectively archive documents with sale dates before 2015 to start)

Click Next

d. Partition on ```saleDate``` only and click Next -> Begin Archiving

After 1-2 minutes, validate that some data was archived by viewing the archive summary page (~1.5MB).
Specifically, examine the ```Min Date Field``` and ```Max Date Field```.


### 5. Configure Custom Criteria Online Archive

Click on 'Delete Archive' from the ... icon on the current archive.
* Please note, the data that was archived has been deleted from cloud object storage now, and is unrecoverable.

Navigate to Configure Online Archive -> Next.

Set the following:

a. Namespace: ```OA-LAB.sales```

b. Custom Critera: 
```
{ 
    "storeLocation": "London" 
}
```

c. Most commonly queried field: ```storeLocation```

Click Next -> Begin Archiving

After 1-2 minutes, validate that some data was archived by viewing the archive summary page (~435KB)

### 6. Connect to the Online Archive

Click on the 'Connect' button on the Online Archive screen

Select 'Shell' and copy the connection string for the 'Connect to Online Archive' option

Navigate to your command line/terminal, and paste the connection string, with the username replaced with your user

Switch to the ```OA-LAB``` database by entering: 
```
use OA-LAB
```

Run the following query to find all sales in this archive:
```
db.sales.aggregate([{$group: { _id: "$storeLocation", count: { $sum: 1}}},{$sort: {count: -1}}])
```

Note that all documents in this collection (~515) have the store location set to ```London```

### 7. Connect to the Online Archive AND Cluster

Enter ```exit``` in the terminal

Click on the 'Connect' button on the Online Archive screen

Select 'Shell' and copy the connection string for the 'Connect to Cluster and Online Archive' option

Navigate to your command line/terminal, and paste the connection string, with the username replaced with your user

Switch to the ```OA-LAB``` database by entering: 
```
use OA-LAB
```

Run the following query to find all sales:
```
db.sales.aggregate([{$group: { _id: "$storeLocation", count: { $sum: 1}}},{$sort: {count: -1}}])
```
Note that documents in this collection have store locations in London and other cities as well


# Data Federation Lab





