# Docker & WSL Hogging C Drive

## Background

My laptop has only 147 GB of space and run Windows 10  as my C Drive. Now any developer who uses many software as part of their development process would agree that this space is grossly inadequate. 

I had to uninstall and constantly clean up my drive to have enough usable disk space to do anything. In last month I reached almost few hundred mb of space and filled out my hard drive. 

I knew something drastic had to be done till I reformat my c drive. I have to aggressively look for the culprits and find alternatives for them. And that is when I  came across TreeSize Free as a solution to scan my drive and list the files and folders that are hogging my space. And the results were phenomenal. I had used such apps in my android phone but have avoided using it on my Windows pc. 

## The Disk Scan

TreeSize Free is very easy to use since I had no disk space left I downloaded a portable version extracted on one of the other spare drives. And ran the program. The UI is very simple. There is a folder icon ``Select Directory`` . There is a drop down under this icon which allows to select the drive and I chose my C drive and started the scan. Then it asks if I would like to scan as Admin which I had to so as to find all the possible files & folders that are hogging my disk space. 

Surprisingly the biggest files were 32 & 8 GB all being vhdx images (virtual disk images used by the hyper-v) being used by Docker desktop which had the wsl integration enabled & wsl itself. Now I had to find a solution how can I move these files to some other drives to get some extra space. It was risky as I was not sure of the corruption risks and making wsl and Docker Desktop unusable which I have been using extensively recently.

## The Solution

After some research I found the solution which helped me a lot. And again SO came to the rescue, with this [link](https://stackoverflow.com/questions/40465979/change-docker-native-images-location-on-windows-10-pro) .

Here are the steps:
- Stop Docker Desktop
-  Stop wsl
 ```
wsl --shutdown
```

- Export Docker Desktop disk to an external Drive
For this I switched to my D drive which has 350 gb of data and ran the following command
```
wsl --export docker-desktop-data docker-desktop-data.tar
```

- Unregister Docker Desktop from wsl
In this step basically I unregisted Docker Desktop as a distribution from wsl which handles the persistence and container management for Docker
```
wsl --unregister docker-desktop-data
```
- Finally re-created the Docker Desktop with wsl with the import of  exported Docker Desktop virtual drive from the new  path. 

```
wsl --import docker-desktop-data D:\docker-new-repo\ docker-desktop-data.tar --version 2
```

`D:\docker-new-repo\` this path signifies that now the vhdx file will be extracted to this new path.

- To check if the import ran fine you can check running the following command
```
wsl -l --all --verbose
sample output:
  docker-desktop         Stopped         2
  docker-desktop-data    Stopped         2
```
### Note
As a side note, before I did this I took backup of my required docker images. Also  removed all images and containers before proceeding which the export and import of the vhdx

To delete all containers including its volumes use,

```
docker rm -vf $(docker ps -a -q)
```

To delete all the images,

```
docker rmi -f $(docker images -a -q)
```

## Finally

I was able to delete the unused vhdx files from C drive and save significant amount of space. Also when I started my Docker Desktop I did not find any issues. This was a quick hack which helped me I am sure this will help others as well. Am yet to remove my wsl but the process is similar. If you need to try this yourself please check the references. And if you want to discuss further please do comment or send me a mail bobquest33(at)gmail(dot)com.

## Reference
- [Docker: How to delete all local Docker images](https://stackoverflow.com/questions/44785585/docker-how-to-delete-all-local-docker-images)
- [Change Docker native images location on Windows 10 Pro](https://stackoverflow.com/questions/40465979/change-docker-native-images-location-on-windows-10-pro)
- [how to move the vhdx of wsl2 to other disk](https://github.com/MicrosoftDocs/WSL/issues/412)
