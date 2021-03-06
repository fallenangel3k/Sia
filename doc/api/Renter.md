Renter API
==========

This document contains detailed descriptions of the renter's API routes. For an
overview of the renter's API routes, see [API.md#renter](/doc/API.md#renter).  For
an overview of all API routes, see [API.md](/doc/API.md)

There may be functional API calls which are not documented. These are not
guaranteed to be supported beyond the current release, and should not be used
in production.

Overview
--------

The renter manages the user's files on the network. The renter's API endpoints
expose methods for managing files on the network and managing the renter's
allocated funds.

Index
-----

| Route                                                         | HTTP verb |
| ------------------------------------------------------------- | --------- |
| [/renter](#renter-get)                                        | GET       |
| [/renter](#renter-post)                                       | POST      |
| [/renter/contracts](#rentercontracts-get)                     | GET       |
| [/renter/downloads](#renterdownloads-get)                     | GET       |
| [/renter/files](#renterfiles-get)                             | GET       |
| [/renter/delete/___*siapath___](#renterdeletesiapath-post)    | POST      |
| [/renter/download/___*siapath___](#renterdownloadsiapath-get) | GET       |
| [/renter/rename/___*siapath___](#renterrenamesiapath-post)    | POST      |
| [/renter/upload/___*siapath___](#renteruploadsiapath-post)    | POST      |

#### /renter [GET]

returns the current settings along with metrics on the renter's spending.

###### JSON Response
```javascript
{
  // Settings that control the behavior of the renter.
  "settings": {
    // Allowance dictates how much the renter is allowed to spend in a given
    // period. Note that funds are spent on both storage and bandwidth.
    "allowance": {  
      // Amount of money allocated for contracts. Funds are spent on both
      // storage and bandwidth.
      "funds": "1234", // hastings

      // Number of hosts that contracts will be formed with.
      "hosts":24,

      // Duration of contracts formed, in number of blocks.
      "period": 6048, // blocks

      // If the current blockheight + the renew window >= the height the
      // contract is scheduled to end, the contract is renewed automatically.
      // Is always nonzero.
      "renewwindow": 3024 // blocks
    }
  },

  // Metrics about how much the Renter has spent on storage, uploads, and
  // downloads.
  "financialmetrics": {
    // How much money, in hastings, the Renter has paid into file contracts
    // formed with hosts. Note that some of this money may be returned to the
    // Renter when the contract ends. To calculate how much will be returned,
    // subtract the storage, upload, and download metrics from
    // ContractSpending.
    "contractspending": "1234", // hastings

    // Amount of money spent on downloads.
    "downloadspending": "5678", // hastings

    // Amount of money spend on storage.
    "storagespending": "1234", // hastings

    // Amount of money spent on uploads.
    "uploadspending": "5678" // hastings
  }
}
```

#### /renter [POST]

modify settings that control the renter's behavior.

###### Query String Parameters
```
// Number of hastings allocated for file contracts in the given period.
funds // hastings

// Duration of contracts formed. Must be nonzero.
period // block height
```

###### Response
standard success or error response. See
[API.md#standard-responses](/doc/API.md#standard-responses).

#### /renter/contracts [GET]

returns active contracts. Expired contracts are not included.

###### JSON Response
```javascript
{
  "contracts": [
    {
      // Block height that the file contract ends on.
      "endheight": 50000, // block height

      // ID of the file contract.
      "id": "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef",

      // Address of the host the file contract was formed with.
      "netaddress": "12.34.56.78:9",

      // Remaining funds left for the renter to spend on uploads & downloads.
      "renterfunds": "1234", // hastings

      // Size of the file contract, which is typically equal to the number of
      // bytes that have been uploaded to the host.
      "size": 8192 // bytes
    }
  ]
}
```

#### /renter/downloads [GET]

lists all files in the download queue.

###### JSON Response
```javascript
{
  "downloads": [
    {
      // Siapath given to the file when it was uploaded.
      "siapath": "foo/bar.txt",

      // Local path that the file will be downloaded to.
      "destination": "/home/users/alice",

      // Size, in bytes, of the file being downloaded.
      "filesize": 8192, // bytes

      // Number of bytes downloaded thus far.
      "received": 4096, // bytes

      // Time at which the download was initiated.
      "starttime": "2009-11-10T23:00:00Z" // RFC 3339 time
    }   
  ]
}
```

#### /renter/files [GET]

lists the status of all files.

###### JSON Response
```javascript
{
  "files": [ 
    {
      // Path to the file in the renter on the network.
      "siapath": "foo/bar.txt",

      // Size of the file in bytes.
      "filesize": 8192, // bytes

      // true if the file is available for download. Files may be available
      // before they are completely uploaded.
      "available": true,

      // true if the file's contracts will be automatically renewed by the
      // renter.
      "renewing": true,

      // Average redundancy of the file on the network. Redundancy is
      // calculated by dividing the amount of data uploaded in the file's open
      // contracts by the size of the file. Redundancy does not necessarily
      // correspond to availability. Specifically, a redundancy >= 1 does not
      // indicate the file is available as there could be a chunk of the file
      // with 0 redundancy.
      "redundancy": 5,

      // Percentage of the file uploaded, including redundancy. Uploading has
      // completed when uploadprogress is 100. Files may be available for
      // download before upload progress is 100.
      "uploadprogress": 100, // percent

      // Block height at which the file ceases availability.
      "expiration": 60000
    }   
  ]
}
```

#### /renter/delete/___*siapath___ [POST]

deletes a renter file entry. Does not delete any downloads or original files,
only the entry in the renter.

###### Path Parameters [(with comments)](/doc/api/Renter.md#path-parameters)
```
// Location of the file in the renter on the network.
*siapath
```

###### Response
standard success or error response. See
[API.md#standard-responses](/doc/API.md#standard-responses).

#### /renter/download/___*siapath___ [GET]

downloads a file to the local filesystem. The call will block until the file
has been downloaded.

###### Path Parameters
```
// Location of the file in the renter on the network.
*siapath     
```

###### Query String Parameters
```
// Location on disk that the file will be downloaded to.
destination 
```

###### Response
standard success or error response. See
[API.md#standard-responses](/doc/API.md#standard-responses).

#### /renter/rename/___*siapath___ [POST]

renames a file. Does not rename any downloads or source files, only renames the
entry in the renter. An error is returned if `siapath` does not exist or
`newsiapath` already exists.

###### Path Parameters
```
// Current location of the file in the renter on the network.
*siapath     
```

###### Query String Parameters
```
// New location of the file in the renter on the network.
newsiapath
```

###### Response
standard success or error response. See
[API.md#standard-responses](/doc/API.md#standard-responses).

#### /renter/upload/___*siapath___ [POST]

uploads a file to the network from the local filesystem.

###### Path Parameters
```
// Location where the file will reside in the renter on the network.
*siapath
```

###### Query String Parameters
```
// Location on disk of the file being uploaded.
source
```

###### Response
standard success or error response. See
[API.md#standard-responses](/doc/API.md#standard-responses).
