BagIt Profiles Specification (DRAFT in progress)
===

Collaborators
---
This draft originally created by members of the Access 2012 Hackfest group: Meghan Currie, Krista Godfrey, Mark Jordan, Nick Ruest, William Wueppelmann, and Dan Chudnov.

Feedback / discussion
---

Please comment on this draft specification in the Digital Curation Google Group, https://groups.google.com/forum/?fromgroups#!forum/digital-curation .

Purpose and background
---

The purpose of the BagIt Profiles Specification is to allow creators and consumers of Bags to agree on optional components of the Bags they are exchanging. Details of the profile are instantiated in a JSON file that both the producing and consuming applications interpret using the conventions described below. The profile file sits at an HTTP URI (e.g., http://foo.example.com/bagitprofiles/profile-bar.json), and can therefore be read by any number of applications creating or consuming Bags:

	                        BagIt Profile JSON file
                                        /       \
                                       v         v
                        Bag creating app 1  -->  Bag consuming app
                        Bag creating app 2


This proposed Specification builds on the sample profile included in the Library of Congress Bagger tool. However, that profile was local to LC and not intended to be implemented widely. The proposed Specification is intended to be an optional extension to the cannonical BagIt spec, and in no way modifies that specification. Like the BagIt spec, this Profile spec is agnostic to the payload stored in a Bag's data directory.

Use cases for BagIt profiles include distributed mass production of Bags, repository or application-specific content ingestion via Bags (e.g. SWORD, Archivematica), and Preservation-as-a-Service.

Workflow
---

The intended workflow for using a BagIt profile is: 

1. The creator of the Bags ensures that Bags it produces meet all of the constraints expressed in the agreed-upon profile file.

2. The consumer of the Bags retrieves the profile file from its URI and validates incoming Bags against it; specifically, it must complete the Bag if fetch.txt is present, validate the complete Bag against the profile, and then validate the Bag against the cannonical BagIt spec. 

Each of these steps may be performed by a separate tool or microservice; in fact, implementers may integrate validation functionality in tools that perform other Bag-processing functions. For example, the completion of a holey Bag might be performed by a more general Bag-processing tool and need not be delegated to a separate validation tool. 

Some profile constraints are fatal: for example, failure to validate 'accept-serialization' or 'accept-version' implies that the rest of the bag is unverifiable and processing should stop. Therefore, the task that checks these two constraints should be performed as early as possible in the workflow. Processing may continue after non-fatal errors in order to generate a comprehensive error report.

Implementation details
---

The following fields make up a BagIt profile. Each field is a top-level JSON key, as illustrated in the examples that follow. LIST in the field definitions indicates that the key can have one or more values, serialized as a JSON array. Itemized values separated by a | indicate allowed options for that field.

1. `BagIt-Profile-Info`:

	A list of tags that describes the profile itself. The following tags are required in this section: "Source-Organization", "External-Description", "Version", and "BagIt-Profile-Identifier". The first two of these tags are taken from the reserved tags defined in the BagIt spec. The value of "Version" contains the version of the profile; the value of "BagIt-Profile-Identifier" is the URI where the profile file is available, and will have the same value as the "BagIt-Profile-Identifier" tag in bag-info.txt (see below). Inclusion of "Contact-Name," "Contact-Phone" and "Contact-Email," as defined in the BagIt spec, is not required but is encouraged.

2. `Bag-Info`:

	Specifies which tags are required, etc. in bag-info.txt. Each tag definition takes two optional parameters: 1) "required" is true or false (default false) and indicates whether or not this tag is required. 2) "values" is a list of acceptable values. If empty, any value is accepted.

	bag-info.txt must contain the tag 'BagIt-Profile-Identifier', with a value of the URI of the JSON file containing the profile. Since Bags complying to a profile must contain this tag, they must also contain a bag-info.txt file.


3. `Manifests-Required`: LIST

	Each manifest file in LIST is required.

4. `Allow-Fetch.txt`: `true`|`false`

	A fetch.txt file is allowed within the bag. Default: `true`

5. `Serialization`: `forbidden`|`required`|`optional`

	Allow, forbid or require serialization of Bags. Default is `optional`.

6. `Accept-Serialization`: LIST

	A list of MIME types acceptable as serialized formats. E.g. "application/zip". If serialization has a value of required or optional, at least one value is needed. If serialization is forbidden, this has no meaning.

7. `Accept-BagIt-Version`: LIST

	A list of BagIt version numbers that will be accepted. At least one version is required.

Examples
---

[bagProfileFoo.json](https://raw.github.com/ruebot/bagit-profiles/master/bagProfileFoo.json)

```json
{
   "BagIt-Profile-Info":{
      "BagIt-Profile-Identifier":"http://www.library.yale.edu/mssa/bagitprofiles/disk_images.json",
      "Source-Organization":"Yale University",
      "Contact-Name":"Mark Matienzo",
      "External-Description":"BagIt profile for packaging disk images",
      "Version":"0.3"
   },
   "Bag-Info":{
      "Bagging-Date":{
         "required":true
      },
      "Source-Organization":{
         "required":true,
         "values":[
            "Simon Fraser University",
            "York University"
         ]
      },
      "Contact-Phone":{
         "required":true
      }
   },
   "Manifests-Required":[
      "md5"
   ],
   "Allow-Fetch.txt":false,
   "Serialization":"required",
   "Accept-Serialization":[
      "application/zip",
      "application/tar"
   ],
   "Accept-BagIt-Version":[
      "0.96",
      "0.97"
   ]
}
```

[bagProfileBar.json](https://raw.github.com/ruebot/bagit-profiles/master/bagProfileBar.json)

```json
{
     "BagIt-Profile-Info":{
      "BagIt-Profile-Identifier":"http://canadiana.org/standards/bagit/tdr_ingest.json",
      "Source-Organization":"Candiana.org",
      "Contact-Name":"William Wueppelmann",
      "Contact-Email":"tdr@canadiana.com",
      "External-Description":"BagIt profile for ingesting content into the C.O. TDR loading dock.",
      "Version":"1.2"
   },
   "Bag-Info":{
      "Source-Organization":{
         "required":true,
         "values":[
            "Simon Fraser University",
            "York University"
         ]
      },
      "Organization-Address":{
         "required":true,
         "values":[
            "8888 University Drive Burnaby, B.C. V5A 1S6 Canada",
            "4700 Keele Street Toronto, Ontario M3J 1P3 Canada"
         ]
      },
      "Contact-Name":{
         "required":true,
         "values":[
            "Mark Jordan",
            "Nick Ruest"
         ]
      },
      "Contact-Phone":{
         "required":false
      },
      "Contact-Email":{
         "required":true
      },
      "External-Description":{
         "required":true
      },
      "External-Identifier":{
         "required":false
      },
      "Bag-Size":{
         "required":true
      },
      "Bag-Group-Identifier":{
         "required":false
      },
      "Bag-Count":{
         "required":true
      },
      "Internal-Sender-Identifier":{
         "required":false
      },
      "Internal-Sender-Description":{
         "required":false
      },
      "Bagging-Date":{
         "required":true
      },
      "Payload-Oxum":{
         "required":true
      }
   },
   "Manifests-Required":[
      "md5"
   ],
   "Allow-Fetch.txt":false,
   "Serialization":"optional",
   "Accept-Serialization":[
      "application/zip"
   ],
   "Accept-BagIt-Version":[
      "0.96"
   ]
}
```

@todo
---

* Add license (CC0/Public Domain?).
* Request feedback from BagIt implementer community, initially in the Digital Curation Google Group.
* Write code to confirm proof-of-concept use cases.
* Formalize specification.
* Write standard libraries to assist in profile validation.
* Establish a public registry of BagIt profiles for common uses, such as ingestion of content into repository platforms.
