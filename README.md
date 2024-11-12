# tflite-builds
Repo for hosting the prebuilt binaries of Tensorflow lite
# Build Process
## On Windows
- Install the prerequistes mentioned in https://www.tensorflow.org/install/source_windows. Following are the versions of the tools that are used while writing this document.
  Python - 3.12
  Bazel - 6.5.0
  Clang+LLVM - 19.1.3 (Make sure to install this in the default directory of C:\Program Files\LLVM\bin, as some dependencies look for this exact path)
- Clone the tensorflow repo from https://github.com/tensorflow/tensorflow and checkout the desired release version. The version used while writing this document is v2.18.0 
- Run the command `python configure.py`, from the tensoflow root folder. It will prompt with some questions. Answer them with default options.
- Run the command `bazel build -c opt --config=win_clang //tensorflow/lite:tensorflowlite`. This should generate the libraries in `bazel-bin\tensorflow\lite` folder.
- Now create a new folder for copying the prebuilt binaries and their corresponding headers. Let's call this folder, `prebuilt`. In this folder, create two subfolders named `include` and `lib`
- Inside the lib folder copy the fles tensorflowlite.dll, tensorflowlite.dll.if.lib and tensorflowlite.pdb
- Inside the include folder we need to copy all the header files recursively from the tensorflow source folder. You could use the below powershell script for copying tflite header files recursively.
```
$SourcePath = (Get-Item "tensorflow\compiler").FullName
$SourceParentPath = (Get-Item "tensorflow").Parent.FullName
$DestinationPath = (Get-Item "prebuilt\include").FullName

Get-ChildItem -Path $SourcePath -Recurse -Filter "*.h" | ForEach-Object {
    # Calculate the relative path from the parent directory of "tensorflow"
    $RelativePath = $_.FullName.Substring($SourceParentPath.Length).TrimStart('\', '/')
    $Destination = Join-Path -Path $DestinationPath -ChildPath $RelativePath

    # Ensure the destination directory exists
    $DestinationDir = Split-Path -Path $Destination -Parent
    if (!(Test-Path -Path $DestinationDir)) {
        New-Item -ItemType Directory -Path $DestinationDir -Force | Out-Null
    }

    # Copy the file to the destination
    Copy-Item -Path $_.FullName -Destination $Destination
}
```
Run the above script twice with source path set as `tensorflow\lite` and `tensorflow\compiler` separately.
- Also copy the folder `flatbuffers` from `bazel-tensorflow\external\flatbuffers\include` to `prebuilt\include` folder.
- Zip the contents of the `prebuilt` folder and upload it as a release.
    
