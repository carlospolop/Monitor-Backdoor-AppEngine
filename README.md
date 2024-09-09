# Monitor & Backdoor Cloud Functions

AppEngine versions generate some data inside a bucket with the format name: `staging.<project-id>.appspot.com`. Inside this bucket, it's possible to find a folder called `ae` that will contain a folder per version of the AppEngine app and inside these folders it'll be possible to find the `manifest.json` file. This file contains a json with all the files that must be used to create the specific version. Moreover, it's possible to find the **real names of the files, the URL to them inside the GCP bucket (the files inside the bucket changed their name for their sha1 hash) and the sha1 hash of each file.**

*Note that it's not possible to pre-takeover this bucket because GCP users aren't authorized to generate buckets using the domain name appspot.com.*

However, with read & write access over this bucket, it's possible to escalate privileges to the SA attached to the App Engine version by monitoring the bucket and any time a change is performed (new version), modify the new version as fast as possible. This way, the container that gets created from this code will execute the backdoored code.

The mentioned attack can be performed in a lot of different ways, all of them start by monitoring the `staging.<project-id>.appspot.com` bucket:

- Upload the complete new code of the AppEngine version to a different and available bucket and prepare a `manifest.json` file with the new bucket name and sha1 hashes of them. Then, when a new version is created inside the bucket, you just need to modify the `manifest.json` file and upload the malicious one.

- Upload a modified `requirements.txt` version that will use a the malicious dependencie code and upate the `manifest.json` file with the new filename, URL and the hash of it.

- Upload a modified `main.py` or `app.yaml` file that will execute the malicious code and update the `manifest.json` file with the new filename, URL and the hash of it.

The script contained inside this folder will monitor the bucket and when a new version is created, it will backdoor it by modifying the `manifest.json` file. Therefore, before performing the attack you need to upload the backdoored files to an accessible bucket and update the `manifest.json` file of this repo with the new filenames, URLs and sha1 hashes. You have some examples of files you could update inside the folder `example_files/`.

```bash
# Installation
python3 -m pip install -r requirements.txt

python3 backdoor_appengine_bucket.py <bucket-name>

#e.g.
python3 backdoor_appengine_bucket.py staging.gcp-labs-3uis1xlx.appspot.com
```
