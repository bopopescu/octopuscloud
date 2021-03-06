# Octopus Cloud Storage System

The [Octopus](http://cloudoctopus.appspot.com/) is a software service, designed to provide a high-availability cloud-based storage solution.

![Octopus logo](documents/OctopusLogo2.png)

The following table provides some information about some of the existing S3-compatible public cloud service offerings and private cloud service solutions:

| Service | Public/Private Cloud | Note |
| ------- | -------------------- | ---- |
| [Amazon S3](http://aws.amazon.com/s3/) | Public | |
| [Cloudian](http://www.cloudian.com/) | Public |  |
| [Connectria Cloud Storage](https://www.mh.connectria.com/rp/order/cloud_storage_index) | Public | It is unclear if this service is still available |
| [Dunkel Cloud Storage](https://www.dunkel.de/s3) | Public |  |
| [Google Cloud Storage](https://cloud.google.com/storage/) | Public |  |
| [Host Europe Cloud Storage](http://www.hosteurope.de/produkte/cloud-storage) | Public | Defunct since end 2014 |
| [HP Helion Public cloud](http://fortune.com/2015/10/21/hp-public-cloud/) | Public | Defunct since January 2016 |
| [Nirvanix](http://www.information-age.com/cloud-adoption-to-soar-as-businesses-pursue-innovation-idc-predicts-123457322/) | Public | Defunct since September 2013 |
| [Apache CloudStack](https://cloudstack.apache.org/) | Private | |
| [Ceph](http://ceph.com/) | Private | |
| [Cumulus (Nimbus)](https://github.com/nimbusproject/nimbus) | Private | |
| [Minio](https://github.com/minio/minio) | Private | |
| [pWalrus](http://www.pdl.cmu.edu/pWalrus/) | Private | Parallel version of Walrus |
| [Riak Cloud Storage](https://github.com/basho/riak_cs) | Private | |
| [S3ninja](https://github.com/scireum/s3ninja) | Private | Emulates the S3 API for development and testing purposes |
| [Swift (OpenStack)](https://github.com/openstack/swift) | Private | |
| [Walrus (Eucalyptus)](https://github.com/eucalyptus/eucalyptus) | Private | |

Octopus' aim is to support multiple different S3-compatible services. Support for S3 and Walrus is implemented now. See the list of already implemented features.

<!---
State October 2016: The web site is offline.
**Web site of Octopus:** [http://cloudoctopus.appspot.com](http://cloudoctopus.appspot.com)
--> 

# Publications

- **Octopus - A Redundant Array of Independent Services (RAIS)**. _Christian Baun, Marcel Kunze, Denis Schwab, Tobias Kurze_. Proceedings of the 3rd International Conference on Cloud Computing and Services Science (CLOSER 2013) in Aachen. SCITEPRESS. ISBN: 978-989-8565-52-5, P.321-328
- [**Redundant Cloud Storage with Octopus**](https://github.com/christianbaun/octopuscloud/blob/master/documents/Octopus_Paper_2011.pdf). _Christian Baun, Marcel Kunze_. This non-published paper from August 2011 summarizes the features and design of Octopus.

# How it works

Octopus is designed to run inside a PaaS like Google’s [AppEngine](https://appengine.google.com), [AppScale](https://github.com/AppScale/appscale) or [typhoonAE](https://sites.google.com/site/gaeasaframework/typhoonae). 

![Octopus deployment options](documents/Octopus_deployment_options.png)

One of the benefits of a cloud platform is that the users don’t need to install the software at client side. 

The users can import their credentials to S3 and Walrus services into Octopus. Octopus checks if a bucket with the naming scheme **octopus_storage-at-<username>** exists. If not, the bucket will be created and the users can upload files - called objects in the S3 world – with one click to the connected storage services into the Octopus bucket. 

The following figure shows the steps to upload an object. After the customers login, his client requests (1) the Octopus website with the HTML form and the list of objects. 

![Octopus object upload](documents/Octopus_upload_object.png)

The object list is requested (2) from the storage services and transferred (3) to Octopus. The synchronicity of the objects is checked (4) by Octopus using checksums. All S3-compatible store a MD5 checksum for each object. These checksums are transferred automatically when a list of objects is requested and they allow to verify if the objects located at the different storage services are synchronized. Any time, when a list of objects is requested, Octopus checks if the objects are still synchronized across the storage services. After the synchronicity check, the web site with the HTML form is transferred (5) to the customers browser. After the customer selected the local file and started the upload with the submit button, the object is transferred (6) to the first storage service. If the upload was successful, a confirmation message is send (7) back to the browser. Step 6 and 7 are repeated for each additional storage service used. 

A drawback of Octopus is that the files that shall be uploaded to the cloud storage services cannot be cached by Octopus itself because files cannot be stored by the applications inside the PaaS. This causes another drawback of Octopus. All files need to be transferred to each connected storage service. If a user has credentials for multiple storage services, the file needs to be transferred from the client (browser) to the storage services one after one.

Because each object is transferred directly from the customers browser to all connected storage services, the amount of data that need to be transferred, increases linear with each additional storage service used. Therefore, the use of multiple storage services leads to disproportionately long transfer times.

Octopus is written in Python and JavaScript. The communication with the S3-compatible storage services is done via boto, a Python interface to the [Amazon Web Services](http://aws.amazon.com/). The user interface is HTML (generated with [Django](https://www.djangoproject.com/)) and some JavaScript ([jQuery](http://jquery.com/)).

# Implemented Features

- Import of credentials for Amazon S3 and Walrus.
- RAID-1 mode. Upload to one or two storage services with a single click.
- Check for synchronicity with help of the MD5 checksums.
- Erase objects inside different storage services with a single click.
- Erase all objects in all storage services with a single click.
- Alter Access Control List (ACL) of objects inside one or two storage services with a single click.

# Next Steps

- Implementation of automatic repair when check for synchronicity failed.
- Currently, each user can import credentials for only one Amazon S3 account and a single Walrus Private Cloud storage service.
- Implement support for Google Blobstore. Objects (called blobs) of max. 2 GB size can be upload into the Blobstore via HTTP POST and then accessed from App Engine applications. Blobstore could be used as a proxy for Octopus to avoid multiple uploads from the browser to the storage services.

![Blobstore as proxy](documents/blobstore_as_proxy.png)

- Google Storage could be used as a proxy for Octopus too because objects inside Google Storage can be accessed from applications running inside the App Engine.
- Implementation of a RAID-5 mode. Benefits would be that no provider has a full (working) copy of the customers data and if a provider is not operational any more, the customers data is still available.

# Challanges and Limitations

- Cumulus does not support uploading objects via POST yet. Maybe future releases have this feature and can be used by Octopus.
- In S3 and Google Storage, the MD5 checksums is enclosed by double quotes. In Walrus they are not.
- If no submit button inside a form is used to upload an object into Walrus, some bytes of garbage data is appended to the object.

