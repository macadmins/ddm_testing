# Tart VM quickstart
To get started testing ddm/mdm changes quickly, here is the TLDR on using tart

- Install using brew:
```
brew install cirruslabs/cli/tart
```
- Grab an ISPW from your favorite source (like [Mr. Macintosh](https://mrmacintosh.com/apple-silicon-m1-full-macos-restore-ipsw-firmware-files-database/) )
- Install from the IPSW:
```
tart create testvm01 --from-ipsw ~/Downloads/UniversalMac_14.5_23F79_Restore.ipsw
```
- Fire it up with a shared folder:
```
mkdir /Users/Shared/tart
tart run testvm01 --dir /Users/Shared/tart
```

Thats basically it! Drop your enrollment profile or whatever else you want into the shared folder, it'll appear as a connected drive in the VM.
