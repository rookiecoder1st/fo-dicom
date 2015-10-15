# Fellow Oak DICOM

[![NuGet](https://img.shields.io/nuget/v/fo-dicom.svg)](https://www.nuget.org/packages/fo-dicom/)
[![NuGet Pre Release](https://img.shields.io/nuget/vpre/fo-dicom.svg)](https://www.nuget.org/packages/fo-dicom/)
[![Build status](https://ci.appveyor.com/api/projects/status/r3yptmhufh3dl1xc?svg=true)](https://ci.appveyor.com/project/anders9ustafsson/fo-dicom)
[![Stories in Ready](https://badge.waffle.io/fo-dicom/fo-dicom.svg?label=ready&title=Ready)](http://waffle.io/fo-dicom/fo-dicom)
[![Join the chat at https://gitter.im/fo-dicom/fo-dicom](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/fo-dicom/fo-dicom?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

### Features
* Core functionality in Portable Class Library
* Targets .NET 4.5 and higher, and Mono
* DICOM dictionary version 2015c
* High-performance, fully asynchronous `async`/`await` API
* JPEG (including lossless), JPEG-LS, JPEG2000, and RLE image compression (.NET only)
* Supports very large datasets with content loading on demand
* Image rendering

### Examples

#### File Operations
```csharp
var file = DicomFile.Open(@"test.dcm");             // Alt 1
var file = await DicomFile.OpenAsync(@"test.dcm");  // Alt 2

var patientid = file.Dataset.Get<string>(DicomTag.PatientID);

file.Dataset.Add(DicomTag.PatientsName, "DOE^JOHN");

// creates a new instance of DicomFile
file = file.ChangeTransferSyntax(DicomTransferSyntax.JPEGProcess14SV1);

file.Save(@"output.dcm");             // Alt 1
await file.SaveAsync(@"output.dcm");  // Alt 2
```

#### Render Image to JPEG
```csharp
var image = new DicomImage(@"test.dcm");
image.RenderImage().Save(@"test.jpg");
```

#### C-Store SCU
```csharp
var client = new DicomClient();
client.AddRequest(new DicomCStoreRequest(@"test.dcm"));
client.Send("127.0.0.1", 12345, false, "SCU", "ANY-SCP");             // Alt 1
await client.SendAsync("127.0.0.1", 12345, false, "SCU", "ANY-SCP");  // Alt 2
```

#### C-Echo SCU/SCP
```csharp
var server = new DicomServer<DicomCEchoProvider>(12345);

var client = new DicomClient();
client.NegotiateAsyncOps();
for (int i = 0; i < 10; i++)
    client.AddRequest(new DicomCEchoRequest());
client.Send("127.0.0.1", 12345, false, "SCU", "ANY-SCP");             // Alt 1
await client.SendAsync("127.0.0.1", 12345, false, "SCU", "ANY-SCP");  // Alt 2
```

#### C-Find SCU
```csharp
var cfind = DicomCFindRequest.CreateStudyQuery(patientId: "12345");
cfind.OnResponseReceived = (DicomCFindRequest rq, DicomCFindResponse rp) => {
	Console.WriteLine("Study UID: {0}", rp.Dataset.Get<string>(DicomTag.StudyInstanceUID));
};

var client = new DicomClient();
client.AddRequest(cfind);
client.Send("127.0.0.1", 104, false, "SCU-AE", "SCP-AE");             // Alt 1
await client.SendAsync("127.0.0.1", 104, false, "SCU-AE", "SCP-AE");  // Alt 2
```

#### C-Move SCU
```csharp
var cmove = new DicomCMoveRequest("DEST-AE", studyInstanceUid);

var client = new DicomClient();
client.AddRequest(cmove);
client.Send("127.0.0.1", 104, false, "SCU-AE", "SCP-AE");             // Alt 1
await client.SendAsync("127.0.0.1", 104, false, "SCU-AE", "SCP-AE");  // Alt 2
```

### 2.0 Breaking Change!
Due to the split into a core Portable Class Library and platform-specific support libraries, manager classes must be initiated before *fo-dicom* can be used in an application. Initialization should be performed using one of the `Dicom.Managers.Setup` overloads, e.g.

    Dicom.Managers.Setup();

In a .NET application, initialization may additionally require selection of the appropriate log and image managers. Available image managers are `WinFormsImageManager.Instance` and `WPFImageManager.Instance`:

    Dicom.Managers.Setup(WinFormsImageManager.Instance);  // or ...
    Dicom.Managers.Setup(WPFImageManager.Instance);

Several log managers are available on .NET, for example `ConsoleLogManager.Instance` and `NLogManager.Instance`:

    Dicom.Managers.Setup(ConsoleLogManager.Instance);  // or ...
    Dicom.Managers.Setup(NLogManager.Instance);        // or ...

To configure both the log and image managers, use the `Dicom.Managers.Setup(LogManager, ImageManager)` overload.

### Contributors
* [Colby Dillion](https://github.com/rcd)
* [Anders Gustafsson](https://github.com/anders9ustafsson) (Cureos AB)
* [Hesham Desouky](https://github.com/hdesouky) (Nebras Technology)
* [Ian Yates](http://github.com/IanYates)
* [Chris Horn](https://github.com/GMZ)
* [Mahesh Dubey](https://github.com/mdubey82)
* [Justin Wake](https://github.com/jwake)
* [captainstark](https://github.com/captainstark)

### License
This library is licensed under the [Microsoft Public License (MS-PL)](http://opensource.org/licenses/MS-PL). See _License.txt_ for more information.
