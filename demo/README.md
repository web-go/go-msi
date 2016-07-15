# demo - go-msi

```sh
vagrant up

# setup go
wget https://storage.googleapis.com/golang/go1.6.2.windows-amd64.msi
vagrant winrm -c "COPY C:\\vagrant\\go1.6.2.windows-amd64.msi C:\\go.msi"
vagrant winrm -c "msiexec.exe /i C:\\go.msi /quiet"
vagrant winrm -c "setx GOPATH C:\\gow\\"
vagrant winrm -c "ls env:GOPATH"
rm go1.6.2.windows-amd64.msi

# setup go-msi
GOOS=windows GOARCH=amd64 go build -o go-msi.exe ../main.go
cp -r ../templates .
vagrant winrm -c "mkdir C:\\go-msi"
vagrant winrm -c "mkdir C:\\go-msi\\templates"
vagrant winrm -c 'Copy-Item C:\\vagrant\\templates\\* -destination C:\\go-msi\\templates\\ -recurse -Force'
vagrant winrm -c 'COPY C:\\vagrant\\go-msi.exe C:\\go-msi\\'
rm -fr templates
rm -fr go-msi.exe

# setup wix
wget -O wix310-binaries.zip http://wixtoolset.org/downloads/v3.10.3.3007/wix310-binaries.zip
unzip wix310-binaries.zip -d wix310
vagrant winrm -c "xcopy /E /I C:\vagrant\wix310 C:\wix310"
vagrant winrm -c "setx PATH \"%PATH%;C:\\wix310\""
rm -fr wix310
rm -fr wix310*zip

# setup the repo
vagrant winrm -c "mkdir C:\\gow"
vagrant winrm -c "mkdir C:\\gow\\src"
vagrant winrm -c "mkdir C:\\gow\\src\\mh-cbon"
vagrant winrm -c "mkdir C:\\gow\\src\\mh-cbon\\github.com"
vagrant winrm -c "mkdir C:\\gow\\src\\mh-cbon\\github.com\\demo"
vagrant winrm -c 'Copy-Item C:\\vagrant\\* -destination C:\\gow\\src\\mh-cbon\\github.com\\demo\\ -recurse -Force'
vagrant winrm -c 'Dir C:\\gow\\src\\mh-cbon\\github.com\\demo\\'

# generate the build
vagrant winrm -c "mkdir C:\\gow\\src\\mh-cbon\\github.com\\demo\\build\\amd64"
vagrant winrm -c 'cmd.exe /c "cd C:\\gow\\src\\mh-cbon\\github.com\\demo\\ && go build -o build\\amd64\\hello.exe hello.go"'
# generate the package
vagrant winrm -c 'cmd.exe /c "cd C:\\gow\\src\\mh-cbon\\github.com\\demo\\ && C:\\go-msi\\go-msi.exe make --msi hello.msi --version 0.0.1 --arch amd64"'
# install software
vagrant winrm -c "msiexec.exe /i C:\\gow\\src\\mh-cbon\\github.com\\demo\\hello.msi /quiet"
vagrant winrm -c "ls env:some"
vagrant winrm -c 'Dir C:\\Program Files\\'
vagrant winrm -c 'Dir C:\\Program Files\\hello'
vagrant winrm -c 'Dir C:\\Program Files\\hello\\assets'
# uninstall software
vagrant winrm -c "msiexec.exe /uninstall C:\\gow\\src\\mh-cbon\\github.com\\demo\\hello.msi /quiet"
```