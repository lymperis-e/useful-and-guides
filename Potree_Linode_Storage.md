<h1>Potree: Store and retrieve PointClouds with Linode Object Storage</h1>

**Prerequisites**

For the following, it is assumed that you have write access to a Linode Object Storage (S3) instance with the identifier: *test-storage*, which is hosted in the *eu-central-1* cluster.

You will also need Cyberduck and s3cmd. It is possible to only use s3cmd, but I have not tested it to figure out the exact commands. s3cmd can be installed on Ubuntu via
```bash
sudo apt-get install s3cmd
```
You will need to configure s3cmd to access your Linode Storage. Follow [this guide](https://www.linode.com/docs/products/storage/object-storage/guides/s3cmd) .

--------------------
**1. Create Directory Structure in Linode S3**   
\
Assuming that PotreeConverter v.2 was used, the files for each point cloud need to be in the same directory. Thus, if you need to store multiple clouds, which will be easily retrievable via a simple url parameter, you need to specify a pseudo-directory structure in S3 (as it is natively not using a hierarchical file system).
\
\
First, you need to create a bucket in S3, via the Linode web interface. Let's say you named it *test-storage*. Then, connect to it with Cyberduck and create a folder in it (let's say *datasets*), using *right-click --> New Folder*. This will create a subdomain of the initial bucket, which will live in 
```
datasets.test-storage.eu-central-1.linodeobjects.com
```

\
Note that it is currently not possible to create multiple folders in the bucket level, but only in its subdomains.
\
Now, in the *datasets* subdomain you can create the folders that will host our point clouds
\
In the same way, using Cyberduck (or s3cmd) you will create a folder for our first point cloud. For example, for a point cloud named *cloud_1*, you will create the folder *cloud_1* in the datasets subdomain. This is the folder that our files will live in, and it is accessibe via: 

```url
https://datasets.test-storage.eu-central-1.linodeobjects.com/cloud_1/
```

------------------
**2. Set CORS policy**   \
\
In order for Potree to be able to access the data in the bucket via *GET RANGE* requests, the CORS policy of the bucket must be set to the following:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
    <CORSRule>
        <AllowedHeader>*</AllowedHeader>
        <AllowedMethod>HEAD</AllowedMethod>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>POST</AllowedMethod>
        <AllowedOrigin>[https://my.frontend.domain.com]</AllowedOrigin>
        <ExposeHeader>Content-Length</ExposeHeader>
        <ExposeHeader>ETag</ExposeHeader>
        <ExposeHeader>x-amz-meta-first-image-name</ExposeHeader>
        <ExposeHeader>x-amz-meta-last-image-name</ExposeHeader>
        <MaxAgeSeconds>0</MaxAgeSeconds>
    </CORSRule>
</CORSConfiguration>
```

Change *[https://my.frontend.domain.com]* to your application / frontend domain name (or just replace it wiht * to allow requests from any domain) and save this into a file, say *cors.xml* 
\
Then, navigate your terminal to the folder you saved the .xml into and run the following command:

```bash
s3cmd setcors cors.xml s3://test-storage.eu-central-1
```

----------------------------
**3. Change Access Policy**

Make sure to change the *Access Policy* of all the folders and items in them to whatever you need (e.g. Public Read). This is easily done via Linode's web interface, though you might want to use s3cmd to automate it for large numbers of files.

--------------------------------------
**4. Load to Potree**


Now upload your point cloud in the subfolder *cloud_1* and load it in Potree with:
```javascript
Potree.loadPointCloud("https://datasets.test-storage.eu-central-1.linodeobjects.com/cloud_1/metadata.json", "Point Cloud Name", async function (e) {
    . . . 
});
```
