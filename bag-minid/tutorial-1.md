# Tutorial #1: Create an abridged bag from assets on a local filesystem

## Introduction
In this tutorial, we will show how one can create an _abridged_ bag that uses `minid` identifers instead of regular
URLs to reference payload files. An abridged (or "holey") bag is a bag where one or more of the payload files
are not distributed with the bag but rather are remotely hosted and referenced by URL in the bag's
[`fetch.txt`](https://tools.ietf.org/html/draft-kunze-bagit-16#section-2.2.3).
Our goal is to create a bag using only remote `minid` references to payload files hosted in cloud storage,
upload the bag to cloud storage as well, and finally mint a `minid` identifier for the bag itself.

This task consists of three stages:

1. [Prepare](#stage_1) the bag's payload files.
2. [Create](#stage_2) the bag, upload it to cloud storage, and publish the bag identifier.
3. [Test](#stage_3) the bag by downloading it, resolving (downloading) the bag's payload, and validating the completed bag.

###### Note:
Certain parts of this tutorial are more demonstrative in nature, and you should be aware that copying and pasting
the command examples below will not necessarily result in the exact output as shown. In particular, generating `minid` identifers
for content will create new identifiers based on your `minid` user account, and uploading files to S3 will be have to be done to a
bucket that you have write access to, and the S3 path locations used in your created identifiers adjusted accordingly.

<a name="stage_1"></a>
## Stage 1: Prepare the bag payload
In this stage, we will generate checksums for each of the bag's payload files, upload each file to cloud storage,
and batch publish identifiers for all of the remote payload files.

To keep things simple, the payload consists of only two files: a README.txt in the root of the payload directory,
and a JPEG image file in an `images` subdirectory. This sample directory can be found [here](assets/tutorial-1/fair-tutorial-bag-minid-input).
```sh
nih-commons:mdarcy[~] ls -lgR fair-tutorial-bag-minid-input/
fair-tutorial-bag-minid-input/:
total 4
drwxrwxr-x. 2 mdarcy  31 Jun  5 23:38 images
-rw-rw-r--. 1 mdarcy 398 Jun  5 22:50 README.txt

fair-tutorial-bag-minid-input/images:
total 320
-rw-rw-r--. 1 mdarcy 324583 Jun  5 22:37 fair-research.jpg
```

We are now going to use a utility routine, `create-rfm-from-filesystem`, from the [`bdbag-utils`](https://github.com/fair-research/bdbag/blob/master/doc/utils.md)
module to generate a [`remote-file-manifest`](https://github.com/fair-research/bdbag/blob/master/doc/config.md#remote-file-manifest).
The `remote-file-manifest` is what `bdbag` uses to create bags that reference payload files remotely. The data from the `remote-file-manifest`
is what is used to generate both the bag's payload `manifests` and `fetch.txt` during bag creation. Conveniently, the `remote-file-manifest` format
is also compatible with the `minid` file manifest format and we will be using the output generated by this command to feed the `minid` CLI command `--batch-register` in a moment.

##### Command line:
```sh
bdbag-utils --debug create-rfm-from-filesystem --checksum md5 --base-url https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input --url-formatter append-path ./fair-tutorial-bag-minid-input ./ftbm-rfm-1.json
```

##### Explanation of arguments:
* The argument `create-rfm-from-filesystem` indicates the sub-command of `bdbag-utils` to run. In this case, it is the routine that creates a
`remote-file-manifest` by recursively scanning a local filesystem directory, calculating checksums for the files it finds, and then generating a target URL for
each file based on the additional URL creation arguments provided.
* The argument `--checksum md5` specifies the hash algorithm to use; in this case, `md5`.
* The argument `--base-url` specifies the base URL path to prepend to each filename when generating the `url` field. In this case, we have specifed
`https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input` as the base cloud path, since we know ahead of time that is the location we intend to
upload the payload files to.
* The argument `--url-formatter append-path` specifies that the existing relative path (including the filename) will be appended to the `--base-url` argument.
* The last two positional arguments, `./fair-tutorial-bag-minid-input` and `./ftbm-rfm-1.json` are the input directory and output file path, respectively.

##### Output:
```
nih-commons:mdarcy[~] bdbag-utils --debug create-rfm-from-filesystem --checksum md5 --base-url https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input --url-formatter append-path ./fair-tutorial-bag-minid-input ./ftbm-rfm-1.json

2018-06-06 22:41:51,332 - INFO - 1 subdirectories found in input directory ./fair-tutorial-bag-minid-input ['images']
2018-06-06 22:41:51,332 - DEBUG - Processing input file ./fair-tutorial-bag-minid-input/README.txt
2018-06-06 22:41:51,332 - DEBUG - Computing [md5] hashes for file [./fair-tutorial-bag-minid-input/README.txt]
2018-06-06 22:41:51,333 - DEBUG - Processing input file ./fair-tutorial-bag-minid-input/images/fair-research.jpg
2018-06-06 22:41:51,333 - DEBUG - Computing [md5] hashes for file [./fair-tutorial-bag-minid-input/images/fair-research.jpg]
2018-06-06 22:41:51,334 - INFO - Successfully created remote file manifest: ./ftbm-rfm-1.json
```

The output file is a `remote-file-manifest` that can be passed directly to `bdbag` when creating a bag.
It can also be used as input to the `minid` command `batch-register`. Notice how the file contains the `md5` checksum in both
_hexadecimal_ and _base64_ encoded forms. This is because the `bagit` specification requires checksum manifests to use the
_hexadecimal_ form, while many HTTP/REST-based object stores (e.g., cloud providers) prefer the _base64_ encoded form.
Also note the relative path preservation in both the `filename` and `url` fields.
```sh
nih-commons:mdarcy[~] cat ./ftbm-rfm-1.json
[
  {
    "filename": "README.txt",
    "length": 398,
    "md5": "b597d3b1126a8db1f09fd70128f28f18",
    "md5_base64": "tZfTsRJqjbHwn9cBKPKPGA==",
    "url": "https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input/README.txt"
  },
  {
    "filename": "images/fair-research.jpg",
    "length": 324583,
    "md5": "51805244fa380526747d937731a75e9c",
    "md5_base64": "UYBSRPo4BSZ0fZN3MadenA==",
    "url": "https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input/images/fair-research.jpg"
  }
]
```

Now that we have our `remote-file-manifest` generated, which includes our checksums, it is time to upload our payload files to our distribution server, in this case Amazon S3.
Note that while the upload process is currently a "manual" process, it is technically possible to add automatic upload support to `bdbag-utils` in the future.

Practically speaking, this upload step can be done at any time. However, performing this step after generating the
`remote-file-manifest` has the benefit of allowing you to specify the `md5` when uploading the file to the distribution server.
When using AWS S3, the benefit of supplying this value during the upload phase is two-fold: 1) the `--Content-MD5` argument tells S3
to perform the equivalent hash calculation after it has received the file (thus ensuring the data integrity of the transfer), and 2)
the `--metadata md5chksum` argument adds the _base64_ encoded `md5` value as an object property which is then returned in the HTTP
header for GET and HEAD requests as `x-amz-meta-md5chksum`.
```sh
nih-commons:mdarcy[~] aws s3api put-object --acl public-read --bucket fair-research --key tutorials/bag-minid/tutorial-1/input/README.txt --body ./fair-tutorial-bag-minid-input/README.txt --content-type "text/plain" --metadata md5chksum=tZfTsRJqjbHwn9cBKPKPGA== --content-md5 tZfTsRJqjbHwn9cBKPKPGA==
{
    "ETag": "\"b597d3b1126a8db1f09fd70128f28f18\""
}
nih-commons:mdarcy[~] aws s3api put-object --acl public-read --bucket fair-research --key tutorials/bag-minid/tutorial-1/input/images/fair-research.jpg --body ./fair-tutorial-bag-minid-input/images/fair-research.jpg --content-type "image/jpeg" --metadata md5chksum=UYBSRPo4BSZ0fZN3MadenA== --content-md5 UYBSRPo4BSZ0fZN3MadenA==
{
    "ETag": "\"51805244fa380526747d937731a75e9c\""
}
```

Next, we are going to create `minid` identifiers for each remote payload file listed in our `remote-file-manifest`. We perform this
step using the `minid` CLI command `batch-register`. This command will take our `remote-file-manifest` as input and generate `minid` identifiers
for each entry in the manifest. The output of this command is the manifest as it was input, but with one exception: the `url` field for each
manifest entry will now be replaced with the registered `minid` identifier for the referenced remote content.

##### Command line:
```sh
minid --batch-register ./ftbm-rfm-1.json > ftbm-rfm-1-minid.json
```
##### Output:
```
nih-commons:mdarcy[~] minid --batch-register ./ftbm-rfm-1.json > ftbm-rfm-1-minid.json
2018-06-06 22:45:27,623 - INFO - Checking if the entity b597d3b1126a8db1f09fd70128f28f18 already exists on the server: http://minid.bd2k.org/minid
2018-06-06 22:45:27,859 - INFO - Creating new identifier
2018-06-06 22:45:29,561 - INFO - Created/updated minid: minid:b9611n
2018-06-06 22:45:29,561 - INFO - Checking if the entity 51805244fa380526747d937731a75e9c already exists on the server: http://minid.bd2k.org/minid
2018-06-06 22:45:29,709 - INFO - Creating new identifier
2018-06-06 22:45:29,406 - INFO - Created/updated minid: minid:b92695
```

Success! Now, let's query them just to be sure:
```
nih-commons:mdarcy[~] minid minid:b9611n
2018-06-06 22:50:06,939 - INFO - Checking if the entity minid:b9611n already exists on the server: http://minid.bd2k.org/minid
Identifier: minid:b9611n
Created by: mdarcy (0000-0003-2280-917X)
Created: 2018-06-06T02:32:51.269349
Checksum: b597d3b1126a8db1f09fd70128f28f18
Status: ACTIVE
Locations:
  mdarcy - https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input/README.txt
Title:
  mdarcy - README.txt
```
```
nih-commons:mdarcy[~] minid minid:b92695
2018-06-06 22:50:43,460 - INFO - Checking if the entity minid:b92695 already exists on the server: http://minid.bd2k.org/minid
Identifier: minid:b92695
Created by: mdarcy (0000-0003-2280-917X)
Created: 2018-06-06T02:32:53.130551
Checksum: 51805244fa380526747d937731a75e9c
Status: ACTIVE
Locations:
  mdarcy - https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input/images/fair-research.jpg
Title:
  mdarcy - fair-research.jpg
```

As you can see, the result output file `ftbm-rfm-1-minid.json` is a `bdbag`-compatible remote file manifest,
with all of the `url` fields replaced with the newly minted `minid` identifiers:
```sh
nih-commons:mdarcy[~] cat ftbm-rfm-1-minid.json
[
  {
    "filename": "README.txt",
    "length": 398,
    "md5": "b597d3b1126a8db1f09fd70128f28f18",
    "md5_base64": "tZfTsRJqjbHwn9cBKPKPGA==",
    "url": "minid:b9611n"
  },
  {
    "filename": "images/fair-research.jpg",
    "length": 324583,
    "md5": "51805244fa380526747d937731a75e9c",
    "md5_base64": "UYBSRPo4BSZ0fZN3MadenA==",
    "url": "minid:b92695"
  }
]
```

<a name="stage_2"></a>
## Stage 2: Create the bag, upload it, and generate an identifier.
Now we're in business! At this point we have everything we need to create a bag from the new `remote-file-manifest`
containing freshly minted `minid` identifier references to our payload files. First, let's create an empty directory that we will
use as the basis for our bag. The directory is empty because our bag contains only remote file references and no local payload.
However, if we wanted to include some local payload at this point we could, just by copying files into the new bag directory.

##### Command line:
```sh
bdbag --checksum md5 --remote-file-manifest ftbm-rfm-1-minid.json --ro-manifest-generate update fair-tutorial-bag-minid
```

##### Explanation of arguments:
* The argument `--checksum md5` specifies the hash algorithm to use; in this case, `md5`. We want to specify the same checksum algorithm we used
when creating the `remote-file-manifest`, that way, any local files that might be added use the same hash algorithm and all payload files are therefore
consistent with regard to the hashing algorithm(s) being used.
    * TECHNICAL NOTE:
With regard to the `--checksum` argument, while it is possible to specify multiple hashing algorithms for bag payload manifests,
the `0.97` and older versions of the `bagit` specification (and `bdbag` and `bagit-python` by association) allowed for subsets of the overall set of
payload files in each manifest. In other words, you could have some files that used `md5` in the payload, and other that used `sha256` (or others) in the
same payload. As of the `bagit-1.0` specification, this is no longer allowed and `bdbag` versions `1.3.0` and greater comply with this restriction.
In other words, if multiple hash algorithms are specified during bag create or update, then a checksum of each specified hash algorithm must be specified for _every_ payload file in the bag.
* The argument `--remote-file-manifest ftbm-rfm-1-minid.json` specifies that the output file `ftbm-rfm-1-minid.json` from our `minid --batch-register`
command should be used as the input `remote-file-manifest`.
* The argument `--ro-manifest-generate update` is a new [function](https://github.com/fair-research/bdbag/blob/master/doc/api.md#generate_ro_manifest)
in the `bdbag-1.4.1` release which causes `bdbag` to generate a basic `bagit-ro` [Research Object](https://github.com/ResearchObject/bagit-ro)
`manifest.json` as a tagfile in the `metadata` tagfile directory. The `update` keyword used with this argument instructs `bdbag` to update any existing
`metadata/manifest.json` file encountered in the bag, as opposed to the `overwrite` keyword which will replace it.
* The final positional argument is the bag path: `fair-tutorial-bag-minid`

##### Output:
```
nih-commons:mdarcy[~] mkdir fair-tutorial-bag-minid
nih-commons:mdarcy[~] bdbag --checksum md5 --remote-file-manifest ftbm-rfm-1-minid.json --ro-manifest-generate update fair-tutorial-bag-minid

2018-06-06 22:56:39,331 - INFO - Generating remote file references from ftbm-rfm-1-minid.json
2018-06-06 22:56:39,331 - INFO - Creating bag for directory /home/mdarcy/fair-tutorial-bag-minid
2018-06-06 22:56:39,331 - INFO - Creating data directory
2018-06-06 22:56:39,332 - INFO - Moving /home/mdarcy/fair-tutorial-bag-minid/tmprN9GBr to data
2018-06-06 22:56:39,332 - INFO - Using 1 processes to generate manifests: md5
2018-06-06 22:56:39,332 - INFO - Generating manifest lines for remote files
2018-06-06 22:56:39,333 - INFO - Writing fetch.txt
2018-06-06 22:56:39,333 - INFO - Creating bagit.txt
2018-06-06 22:56:39,333 - INFO - Creating bag-info.txt
2018-06-06 22:56:39,333 - INFO - Creating /home/mdarcy/fair-tutorial-bag-minid/tagmanifest-md5.txt
2018-06-06 22:56:39,335 - INFO - Created bag: /home/mdarcy/fair-tutorial-bag-minid
2018-06-06 22:56:39,336 - INFO - Auto-generating RO manifest: overwrite existing.
2018-06-06 22:56:39,338 - INFO - Creating /home/mdarcy/fair-tutorial-bag-minid/tagmanifest-md5.txt
```

Success! Now, let's have a look in that bag...
```sh
nih-commons:mdarcy[~] ls -lgR fair-tutorial-bag-minid
fair-tutorial-bag-minid:
total 20
-rw-rw-r--. 1 mdarcy 298 Jun  6 22:56 bag-info.txt
-rw-rw-r--. 1 mdarcy  55 Jun  6 22:56 bagit.txt
drwxrwxr-x. 2 mdarcy   6 Jun  6 22:56 data
-rw-rw-r--. 1 mdarcy  83 Jun  6 22:56 fetch.txt
-rw-rw-r--. 1 mdarcy 114 Jun  6 22:56 manifest-md5.txt
drwxrwxr-x. 2 mdarcy  27 Jun  6 22:56 metadata
-rw-rw-r--. 1 mdarcy 238 Jun  6 22:56 tagmanifest-md5.txt

fair-tutorial-bag-minid/data:
total 0

fair-tutorial-bag-minid/metadata:
total 4
-rw-rw-r--. 1 mdarcy 977 Jun  6 22:56 manifest.json
```

Nice, `fetch.txt` looks good!
```sh
nih-commons:mdarcy[~] cat fair-tutorial-bag-minid/fetch.txt
minid:b9611n    398     data/README.txt
minid:b92695    324583  data/images/fair-research.jpg
```

So does `manifest-md5.txt`.
```sh
nih-commons:mdarcy[~] cat fair-tutorial-bag-minid/manifest-md5.txt
b597d3b1126a8db1f09fd70128f28f18  data/README.txt
51805244fa380526747d937731a75e9c  data/images/fair-research.jpg
```

What about `metadata/manifest.json` -- what got generated? As you can see, it is a bare-bones RO manifest. But, the good part is that the
`aggregates` section was generated automatically by introspecting the bag structure. Also, some of the basic provenance fields got populated
the same way, via `bag-info.txt`.
```sh
nih-commons:mdarcy[~] cat fair-tutorial-bag-minid/metadata/manifest.json
{
    "@context": [
        "https://w3id.org/bundle/context"
    ],
    "@id": "../",
    "aggregates": [
        {
            "bundledAs": {
                "filename": "README.txt",
                "folder": "../data/",
                "uri": "urn:uuid:ab664832-43f0-4cbe-85ee-552c98f9610d"
            },
            "mediatype": "text/plain",
            "uri": "minid:b9611n"
        },
        {
            "bundledAs": {
                "filename": "fair-research.jpg",
                "folder": "../data/images",
                "uri": "urn:uuid:ab664832-43f0-4cbe-85ee-552c98f9610d"
            },
            "mediatype": "image/jpeg",
            "uri": "minid:b92695"
        }
    ],
    "annotations": [],
    "authoredBy": {
        "name": "mdarcy",
        "orcid": "http://orcid.org/0000-0003-2280-917X"
    },
    "authoredOn": "2018-06-06T23:06:57+00:00",
    "createdBy": {
        "name": "BDBag version: 1.4.1 (Bagit version: 1.6.4)",
        "uri": "https://github.com/fair-research/bdbag"
    },
    "createdOn": "2018-06-06T23:06:57+00:00"
}
```

OK, since everything is looking correct, let's go ahead and archive (serialize) the bag:
```
nih-commons:mdarcy[~] bdbag --archive zip fair-tutorial-bag-minid

2018-06-06 23:45:03,488 - INFO - The directory /home/mdarcy/fair-tutorial-bag-minid is already a bag.
2018-06-06 23:45:03,488 - INFO - Validating bag structure: /home/mdarcy/fair-tutorial-bag-minid
2018-06-06 23:45:03,490 - INFO - The directory /home/mdarcy/fair-tutorial-bag-minid is a valid bag structure
2018-06-06 23:45:03,490 - INFO - Archiving bag (zip): /home/mdarcy/fair-tutorial-bag-minid
2018-06-06 23:45:03,491 - INFO - Created bag archive: /home/mdarcy/fair-tutorial-bag-minid.zip
```

Now let's upload it to S3. In this case, we are less concerned about providing a checksum to S3 because next we are going
to create a `minid` identifier for this bag and that process will create a SHA256 hash of the serialized bag and then associate the
hash value with the `minid` identifier. However, if you simply want to ensure the data integrity of the upload to S3, you could just run
`openssl md5 -binary ./fair-tutorial-bag-minid.zip | base64` and specify the output value as the `--Content-MD5` argument to the
`aws s3api put-object` command.
```sh
nih-commons:mdarcy[~] aws s3api put-object --acl public-read --bucket fair-research --key tutorials/bag-minid/tutorial-1/output/fair-tutorial-bag-minid.zip --body ./fair-tutorial-bag-minid.zip
{
    "ETag": "\"230157df2f1c3886e18a04988c57150f\""
}
```

Finally, let's create a minid for the bag archive:
```
nih-commons:mdarcy[~] minid --register --title "Fair Research BDBag/Minid Tutorial #1 - Example Bag"  fair-tutorial-bag-minid.zip --locations https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/output/fair-tutorial-bag-minid.zip
2018-06-07 00:31:14,334 - INFO - Computing checksum for fair-tutorial-bag-minid.zip using <sha256 HASH object @ 0x7f97f95d7828>
2018-06-07 00:31:14,334 - INFO - Checking if the entity 70b8bccc69680f9b0bb53db562fdf3bbd84bb4d0f2f21517fa428fb3d38d41b3 already exists on the server: http://minid.bd2k.org/minid
2018-06-07 00:31:14,522 - INFO - Creating new identifier
2018-06-07 00:31:16,244 - INFO - Created/updated minid: minid:b90986
```

Let's verify that minid before we continue:
```
nih-commons:mdarcy[~] minid minid:b90986
2018-06-07 00:31:36,146 - INFO - Checking if the entity minid:b90986 already exists on the server: http://minid.bd2k.org/minid
Identifier: minid:b90986
Created by: mdarcy (0000-0003-2280-917X)
Created: 2018-06-07T00:30:32.993708
Checksum: 70b8bccc69680f9b0bb53db562fdf3bbd84bb4d0f2f21517fa428fb3d38d41b3
Status: ACTIVE
Locations:
  mdarcy - https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/output/fair-tutorial-bag-minid.zip
Title:
  mdarcy - Fair Research BDBag/Minid Tutorial #1 - Example Bag
```
<a name="stage_3"></a>
## Stage 3: Download the bag, resolve it's payload, and validate.
We're in the home stretch! Now its time to see the fruits of our labor in action. First, let's download the bag:
```
nih-commons:mdarcy[~] wget https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/output/fair-tutorial-bag-minid.zip
--2018-06-07 00:32:56--  https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/output/fair-tutorial-bag-minid.zip
Resolving fair-research.s3.amazonaws.com (fair-research.s3.amazonaws.com)... 52.218.197.10
Connecting to fair-research.s3.amazonaws.com (fair-research.s3.amazonaws.com)|52.218.197.10|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1934 (1.9K) [binary/octet-stream]
Saving to: ‘fair-tutorial-bag-minid.zip’

fair-tutorial-bag-minid.zip       100%[================================================================>]   1.89K  --.-KB/s    in 0s

2018-06-07 00:32:56 (89.2 MB/s) - ‘fair-tutorial-bag-minid.zip’ saved [1934/1934]
```

Next, extract the bag and resolve (download) all of the remotely referenced payload files.
```
nih-commons:mdarcy[~] bdbag ./fair-tutorial-bag-minid.zip

2018-06-07 00:35:36,582 - INFO - Extracting ZIP archived bag file: /home/mdarcy/fair-tutorial-bag-minid.zip
2018-06-07 00:35:36,584 - INFO - File /home/mdarcy/fair-tutorial-bag-minid.zip was successfully extracted to directory /home/mdarcy/fair-tutorial-bag-minid

nih-commons:mdarcy[~] bdbag --fetch all ./fair-tutorial-bag-minid

2018-06-07 00:35:51,623 - INFO - Attempting to resolve remote file references from fetch.txt...
2018-06-07 00:35:51,626 - INFO - Attempting to resolve minid:b9611n into a valid set of URLs.
2018-06-07 00:35:52,054 - INFO - The identifier minid:b9611n resolved into the following locations: [https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input/README.txt]
2018-06-07 00:35:52,054 - INFO - Attempting GET from URL: https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input/README.txt
2018-06-07 00:35:52,262 - INFO - File [/home/mdarcy/fair-tutorial-bag-minid/data/README.txt] transfer successful. 0.389 KB transferred. Elapsed time: 0:00:00.000322.
2018-06-07 00:35:52,263 - INFO - Attempting to resolve minid:b92695 into a valid set of URLs.
2018-06-07 00:35:52,545 - INFO - The identifier minid:b92695 resolved into the following locations: [https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input/images/fair-research.jpg]
2018-06-07 00:35:52,545 - INFO - Attempting GET from URL: https://fair-research.s3.amazonaws.com/tutorials/bag-minid/tutorial-1/input/images/fair-research.jpg
2018-06-07 00:35:52,907 - INFO - File [/home/mdarcy/fair-tutorial-bag-minid/data/images/fair-research.jpg] transfer successful. 316.976 KB transferred. Elapsed time: 0:00:00.274359.
2018-06-07 00:35:52,907 - INFO - Fetch complete. Elapsed time: 0:00:01.281004
```

Yes! We got all of the payload successfully. But does it validate?
```
nih-commons:mdarcy[~] bdbag --validate full ./fair-tutorial-bag-minid

2018-06-07 01:17:26,970 - INFO - Validating bag: /home/mdarcy/fair-tutorial-bag-minid
2018-06-07 01:17:26,972 - INFO - Verifying checksum for file /home/mdarcy/fair-tutorial-bag-minid/fetch.txt
2018-06-07 01:17:26,972 - INFO - Verifying checksum for file /home/mdarcy/fair-tutorial-bag-minid/data/images/fair-research.jpg
2018-06-07 01:17:26,973 - INFO - Verifying checksum for file /home/mdarcy/fair-tutorial-bag-minid/data/README.txt
2018-06-07 01:17:26,973 - INFO - Verifying checksum for file /home/mdarcy/fair-tutorial-bag-minid/metadata/manifest.json
2018-06-07 01:17:26,973 - INFO - Verifying checksum for file /home/mdarcy/fair-tutorial-bag-minid/bagit.txt
2018-06-07 01:17:26,973 - INFO - Verifying checksum for file /home/mdarcy/fair-tutorial-bag-minid/bag-info.txt
2018-06-07 01:17:26,973 - INFO - Verifying checksum for file /home/mdarcy/fair-tutorial-bag-minid/manifest-md5.txt
2018-06-07 01:17:26,974 - INFO - Bag /home/mdarcy/fair-tutorial-bag-minid is valid
```

Success! We're done! High Fives all around!


