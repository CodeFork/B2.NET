# B2.NET
[![ghit.me](https://ghit.me/badge.svg?repo=coryrwest/B2.NET)](https://ghit.me/repo/coryrwest/B2.NET) [![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

B2.NET is a C# client for the [Backblaze B2 Cloud Storage](https://secure.backblaze.com/b2/) service.

B2.NET is still in Beta, so use it in production at your own risk.

[B2 Documentation](https://www.backblaze.com/b2/docs/)

## Features

*  Full implementation of the B2 REST API
*  Experimental support for file FriendlyURL's
*  UFT-8 and Url Encoding support
*  Full Async support
*  Full test coverage
*  Targets .NET 4.5 and .NET Standard 1.5
*  NOTE: There is currently no plans to support the new Key's API's

## nuget
[nuget package](https://www.nuget.org/packages/B2Net/)

```
Install-Package B2Net
```

## Install

Stick the B2Net.dll in your project.

## Usage
```csharp
// the B2Client will default to the bucketId provided here
// for all subsequent calls if you set PersistBucket to true.
var options = new B2Options() {
	AccountId = "YOUR ACCOUNT ID",
	ApplicationKey = "YOUR APPLICATION KEY",
	BucketId = "OPTIONAL BUCKET ID",
	PersistBucket = true/false
};
var client = new B2Client(Options);
```

### Authorize
There are two ways to Authorize a B2Client. The old way is slightly more verbose and there is no clear benefit to using it over the 
static method. The `options` object returned from the Authorize method will contain the authorizationToken necessary for subsequent 
calls to the B2 API. This will automatically be handled by the library when necessary. You do not have to keep this object around.
The `options` object requires AccountID and ApplicationKey to authorize.
```csharp
// New, cleaner way to Authorize using the static `Authorize` method.
var client = new B2Client(B2Client.Authorize(options));
// OR
var client = new B2Client(B2Client.Authorize("ACCOUNTID", "APPLICATIONKEY"));
// OR
var client = new B2Client("ACCOUNTID", "APPLICATIONKEY", "REQUESTTIMEOUT");
```

The old way:
```csharp
var client = new B2Client("ACCOUNTID", "APPLICATIONKEY");
var options = await client.Authorize();
```

### List Buckets
```csharp
var client = new B2Client("ACCOUNTID", "APPLICATIONKEY");
var bucketList = await client.Buckets.GetList();
// [
//   { BucketId: "",
//     BucketName: "",
//     BucketType: "",
//     BucketInfo: Dictionary<string,string> }
// ]
```

### Create a Bucket
```csharp
var client = new B2Client("ACCOUNTID", "APPLICATIONKEY");
var bucket = await client.Buckets.Create("BUCKETNAME", "OPTIONAL_BUCKET_TYPE");
// { BucketId: "",
//   BucketName: "",
//   BucketType: "" }
```

### Create a Bucket with options
```csharp
var client = new B2Client("ACCOUNTID", "APPLICATIONKEY");
var bucket = await client.Buckets.Create("BUCKETNAME", new B2BucketOptions() {
	CacheControl = 300,
	LifecycleRules = new System.Collections.Generic.List<B2BucketLifecycleRule>() {
		new B2BucketLifecycleRule() {
			DaysFromHidingToDeleting = 30,
			DaysFromUploadingToHiding = null,
			FileNamePrefix = ""
		}
	}
});
// { BucketId: "",
//   BucketName: "",
//   BucketType: "",
//   BucketInfo: Dictionary<string,string>,
//   LifecycleRules: List<B2BucketLifecycleRule> }
```

### Update a Bucket
```csharp
var client = new B2Client("ACCOUNTID", "APPLICATIONKEY");
var bucket = await client.Buckets.Update("BUCKETID", "BUCKETYPE");
// { BucketId: "",
//   BucketName: "",
//   BucketType: "" }
```

### Update a Bucket with options
```csharp
var client = new B2Client("ACCOUNTID", "APPLICATIONKEY");
var bucket = await client.Buckets.Update("BUCKETID", new B2BucketOptions() {
	CacheControl = 300,
	LifecycleRules = new System.Collections.Generic.List<B2BucketLifecycleRule>() {
		new B2BucketLifecycleRule() {
			DaysFromHidingToDeleting = 30,
			DaysFromUploadingToHiding = null,
			FileNamePrefix = ""
		}
	}
});
// { BucketId: "",
//   BucketName: "",
//   BucketType: "",
//   BucketInfo: Dictionary<string,string>,
//   LifecycleRules: List<B2BucketLifecycleRule> }
```

#### Bucket Types
```
allPrivate
allPublic
```

### Delete a Bucket
```csharp
var client = new B2Client("ACCOUNTID", "APPLICATIONKEY");
var bucket = await client.Buckets.Delete("BUCKETID");
// { BucketId: "",
//   BucketName: "",
//   BucketType: "" }
```

### Get a list of files
```csharp
var client = new B2Client("ACCOUNTID", "APPLICATIONKEY");
var fileList = await client.Files.GetList("BUCKETID", "FILENAME");
// Using optional prefix and delimiter
var fileList = await client.Files.GetList("BUCKETID", "FILENAME", prefix: "PREFIX", delimiter: "DELIMITER");
// {
//   NextFileName: "",
//   [
//     { FileId: "",
//       FileName: "",
//       ContentLength: "", 
//       ContentSHA1: "", 
//       ContentType: "" }
//   ]
// }
```

### Upload a file
```csharp
var client = new B2Client(options);
await client.Authorize();
var uploadUrl = Client.Files.GetUploadUrl("BUCKETID").Result;
var file = client.Files.Upload("FILEDATABYTES", "FILENAME", uploadUrl, "AUTORETRY", "BUCKETID", "FILEINFOATTRS").Result;
// { FileId: "",
//   FileName: "",
//   ContentLength: "", 
//   ContentSHA1: "", 
//   ContentType: "",
//   FileInfo: Dictionary<string,string> }
```

### Download a file by id
```csharp
var client = new B2Client(B2Client.Authorize(options));
var file = client.Files.DownloadById("FILEID").Result;
// { FileId: "",
//   FileName: "",
//   ContentLength: "", 
//   ContentSHA1: "", 
//   ContentType: "",
//   FileData: byte[],
//   FileInfo: Dictionary<string,string> }
```

### Download a file by name
```csharp
var client = new B2Client(options);
await client.Authorize();
var file = client.Files.DownloadName("FILENAME", "BUCKETNAME").Result;
// { FileId: "",
//   FileName: "",
//   ContentLength: "", 
//   ContentSHA1: "", 
//   ContentType: "",
//   FileData: byte[],
//   FileInfo: Dictionary<string,string> }
```

### Get versions for a file
```csharp
var client = new B2Client(B2Client.Authorize(options));
var file = client.Files.GetVersions("FILENAME", "FILEID").Result;
// {
//   NextFileName: "",
//   NextFileId: "",
//   [
//     { FileId: "",
//       FileName: "",
//       Action: "", 
//       Size: "", 
//       UploadTimestamp: "" }
//   ]
// }
```

### Delete a file version
```csharp
var client = new B2Client(options);
await client.Authorize();
var file = client.Files.Delete("FILEID", "FILENAME").Result;
// { FileId: "",
//   FileName: ""}
```

### Hide a file version
```csharp
var client = new B2Client(B2Client.Authorize(options));
var file = client.Files.Hide("FILEID", "BUCKETID").Result;
// { FileId: "",
//   FileName: "",
//   Action: "",
//   Size: "",
//   UploadTimestamp: ""}
```

### Get info for a file
```csharp
var client = new B2Client(options);
await client.Authorize();
var file = client.Files.GetInfo("FILEID").Result;
// { FileId: "",
//   FileName: "",
//   ContentSHA1: "",
//   BucketId: "",
//   ContentLength: "",
//   ContentType: "",
//   FileInfo: Dictionary<string,string> }
```

### Large File API
See the Large File tests for usage details.

## Release Notes

*  0.5.0  Large file management apis (version mix up as well)
*  0.4.9  static Authorize
*  0.4.6  Configureable HttpClient timeout
*  0.4.5  File prefix and delimiter support
*  0.4.0  Added Large File support
*  0.2.5  Support for .NET 4.5 and .NET Standard 1.5 and FriendlyURL's for files
*  0.1.92 Fixed Lifecycle Rules null issue.
*  0.1.9  Added Lifecycle Rules and Cache Control to bucket creation and updating.
*  0.1.81 Switch targeting to netstandard1.3 and updated B2 endpoint
*  0.1.7  Merged changes for file name encoding and maxFileCount from tomvoros
*  0.1.6  Started URL Encoding file names.
*  0.1.5  File info attributes support.
*  0.1.0  Initial Alpha release.


## Running the tests
From the src directory run
```
dotnet pack -o ..\tests\
```

From the tests directory run 
```
dotnet restore
dotnet test
```
