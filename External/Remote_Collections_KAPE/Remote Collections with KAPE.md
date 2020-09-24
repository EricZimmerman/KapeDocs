

# Remote Collections with KAPE

Triage style collection of remote system data is a fact of life these days and KAPE is an excellent tool to preform this task. *Remote* collections have different meanings depending on the context. In one case, it could mean a system on the corporate LAN, but in another, it could mean a remote system that is only available over the Internet. To get started, let's start with some basics and then show how we can apply the concepts of a LAN collection to the increasingly common situation of collection over the Internet. 

In this article, we are assuming that you have a basic understanding of KAPE terms like **targets** and **modules**. Although we are showing the command line versions of these commands, the concepts will work equally well the the GUI version of KAPE.

## UNC Paths

UNC stands for "Universal Naming Convention," and this is not just the home of the North Carolina Tar Heels. UNC is a filename format that is used to specify the location of files, folders, and resources on a local-area network (LAN). An example of a UNC address for a file may look something like this:

```
\\server-name\directory\filename
```

A common thing you have probably seen before is using a UNC path to create a mapped network drive so we can access the network share as a drive letter.

## Share the C Drive on the Target 

To use KAPE in this manner some setup is required. We need to be able to access the remote system's C drive, meaning the the drive must have sharing enabled. In our examples in this paper, we are making an assumption that you have some access to the remote computer to enable the shares. In a simple Workgroup environment, that would involve these steps.

1.  Select the C Drive On the Remote System
2. Right Click the C Drive and select properties 
3. Click the **"Sharing"** Tab
4. Click the **"Advanced Sharing"** button
5. Check the **"Share this folder"**
6. Click the **"Permissions"** button
7. Check the **"Full Control"** box
8. Click the **"Ok**" button
9. Click the **"Ok"** button on the Advanced Sharing dialog box
10. Click the **"Close"** button 

Click this [link](https://www.tomshardware.com/news/how-to-share-drives-windows-pc,36936.html) for an article on Sharing Drives in Windows.

Click this [link](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/create_shares_steps.png) for a graphical step by step on Sharing Drives in Windows.



Once the share has been created on the target we will create a mapped drive on the collection system to the Target share so we can access it as a drive letter.

1. Open a CMD or PowerShell session as administrator.

2. Ping the target to confirm network connectivity. Either host name or IP address is fine. 

   ```
   ping 192.168.96.162
   ```

   or 

   ```
   ping target -1
   ```

3. Issue the following command to map the target's shared drive a drive letter on the collection system.

   ```
   net use k: \\target-1\c /user:target-1\sansdfir
   ```

   Here, we are assigning the drive letter "k" to the UNC path \\\target-1\c using the fully qualified user name on the target. 

4. Issue the "net use" command to verify that the drive was mapped correctly. The "net use" command without options will show all the current mappings.

   ```
   net use 
   ```

5. Issue the "dir" command to verify that we can access the UNC path via the drive letter. We can use just the drive letter to access that target share.

## Run KAPE using the mapped drive to the Target

When using KAPE in a remote collection, a natural first thought is how to reference that remote system in the KAPE command line. Using a UNC path or mapped drive as the --tsource parameter is how we do that.  Here is an example of what that command might look like. The first example uses the UNC path and the second uses a mapped drive (that is based on a UNC path).

```
kape.exe --tsource \\target-1\c --target LnkFilesAndJumpLists --tdest c:\kape_out\test 
```

```
kape.exe --tsource K:\c --target LnkFilesAndJumpLists --tdest c:\kape_out\tdest 
```

The two commands above collect from the remote target and saves the files matching the target definition (--target) to the \kape_out\tdest folder on the collection system's local C drive.

Let's give this a test drive.

![kape_from_server_lnk](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/kape_from_server_lnk.png)

The Copy Log from this run of KAPE shows what files were collected. There are two other logs created, skipped log and console log. Skipped log shows you the files that met the target search criteria but were not copied for some reason (typically they were deduped based on the SHA-1 of a file already copied by KAPE) and the console log show the same thing that is in the screenshot above. 

![from_srvr_copy_log](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/from_srvr_copy_log.png)

In the Copy Log we see lnk files like we would expect from the command line we just executed, so it looks like we got what we were expecting. Important point of interest, volume shadow copies can't be copied over a UNC path. 

Now lets try a different target. Let's grab some Registry files. We are selecting these files to demonstrate another important limitation of using a UNC path to the target system in addition to a VSC access limitation. On a running system, the Registry files are locked by Windows and because of this, cannot be copied.

```d
kape.exe --tsource \\remote-target-name\c --target RegistryHives --tdest c:\kape_out\test 
```

![kape_from_server_reg](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/kape_from_server_reg.png)

In this screenshot we see two things. The first is that KAPE is deferring the copy of certain files: these are the **in-use** files. These files will be copied later in the process using a different method that uses raw disk reads to acquire the data. Raw disk access gets around the problem of the files being in use. Unfortunately, Windows does not allow this type of copy across a UNC path.  Bummer. It is important to point out that this is not a KAPE error or limitation of KAPE, this is a file system limitation.  Whatever the reason, it keeps us from collecting what we need using this approach.  Lets look at a different approach, still using UNC paths, that will work on these locked files as well as getting us access to any volume shadow copies (assuming they exist).

## Run KAPE from the Target 

Lets looks at this problem from a different angle. The issue is that attempting to copy the locked files from the target across the UNC path with a raw copy. What if we process the files on the target and copy the resulting files, acquired via raw disk reads, across the UNC path? The key here is running KAPE **from** a UNC share, as opposed to trying to access a file system via UNC path. This also gets us the additional benefit of not having to copy any files down to the target machine, which minimizes the footprint we leave behind on the target. There will still be footprints (artifacts) because of the program execution. The will be a Prefetch file created or updated and other Window Registry entries. But, we won't overwrite anything that might be in unallocated space. More on that in a minute. 

Here is an overview of how this works.

1. From the collection system, share the folder on the collection system that contains the kape.exe executable. We are going to map that share to a drive letter on the target. Basically, we are doing the opposite of what we did earlier to share the Target's hard drive.  We are also going to save the results of our collection to subfolder under the kape folder on the collection system because we are lazy. This way we don't need to create two shares.

2. On the target system, use the **"net use"** command as we did above to map the kape folder on the collection system to a drive letter on the Target system.  Note that this is an optional step and we could simply execute KAPE without mapping the drive (i.e. \\server\share\kape.exe ...)

   ```
   net use k: \\kape-server-vm\triage /user:kape-server-vm\analyst
   ```

3. Verify the share and the mapped drive on the target as we did for the server above.

![netuse_from_target](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/netuse_from_target.png)

Now we are going to run a very similar kape command to collect the locked Registry Files from the Target and save the collected files to the collection system. The approach does not write any collected data or temp files to the target. This is because the actual kape.exe is kept on the collection system and the collected data is written directly over the UNC path to the collection system without ever being written to the target machine. Oh, and do not forget, and we can grab the Volume Shadow Copies too with the --vss switch!

```
k:\kape\kape.exe --tsource C --target RegistryHives --tdest k:\kape_out\tdest --vss
```

![kape_from_target_reg](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/kape_from_target_reg.png)



## Be careful - KAPE will write files to the Target

In certain situations, when running KAPE from a target system, KAPE will write it's collection files to the Target system even if the ultimate destination is somewhere off the target system. These files maybe just temporary and will be deleted after the files have been sent to another destination. KAPE has some great features that allow target files to be written to an SFTP server, S3 compliant services, and Azure. KAPE can even be the SFTP server itself. For these features to work, these files must first be written somewhere and the location is often the Target system itself. 

Of course, we could just as easily write the collected target files to the UNC share and then transmit it to remote storage, but this can be slower than writing the data locally. It is nice to have options tho!

The easiest way to see if this situation is going to occur is to look at the command line you are going to run. if the **--tdest** option or the **--mdest** option is a location on the Target's OS drive or other local hard drive, you are going to have the collected files first written to the target. Lets look at a command line run from the target for sending the collected files to an SFTP server. 

```
kape.exe --tsource C: --target !SANS_Triage --tdest C:\temp\tout --scs 104.248.94.196 --scp 22 --scu kape-ssh --scpw "KAP3g0at" --vhdx %m_kape_collect 
```

What's happening in the command line above? First thing is that we have used a **--tdest** option that is a location on the Target system.  Additionally, the creation of the VHDX, which is required when we use the **--scs** option, and the subsequent zipping up of the VHDX will write to the Target system. Technically, any of the container options work, but VHDX is the most common one. We are doing a triage collection, files collected are defined by the !SANS_Triage target, the files are going to be sent to an SFTP server at 104.248.94.196. The files will be packaged up in a VHDX and further zipped up to reduce the transfer time. This is very useful option of KAPE as it allows us to get the collected files out to a secure server where the analyst team can get to work on them right away. 

I don't want to make too big of a deal out of this but there is a risk in running the command above. The files will first be written to the Target system. This may not matter or it may be a trade off. The trade off being that we are betting that getting the files to a secure location is far more beneficial than any risk of potentially losing artifacts in unallocated space. There may be no need to take a full image of the target (or no way of getting a full forensic image of this system) so we don't worry about. The point is that you are knowingly making this decision. 

But, and I have been in this situation, we don't have any reason to think that unallocated space is a factor until after we analyze the triage data and we do in fact have the option of doing a full image. If this is the case what can we do to avoid the potential of overwriting evidence in unallocated? We will cover one solution in the next section. 

## Remote KAPE Collections across the Internet

The techniques of running the kape.exe executable from a network share and writing the collected files to a network share are simple but very powerful. We have shown this technique using very narrowly focused targets but it could be an entire triage collection.  It was simple to create the shares because in our demo we were in a LAN situation. We had various ways to create the shares through something like PowerShell or even logging into the target system via RDP. How do we do this over Internet, over a WAN?

The technique I'm going to describe is to use a free, open source service called ZeroTier One. Basically ZeroTier allows us to create a Software Defined Wide Area Network (SD-WAN) in minutes. The ZeroTier network that we are going to define will work exactly like the examples that we have shown above. 

Yes, we are going to install a very small application on the target but we are not going to write KAPE or any of the collected files to the target. This is a key point. There have been some very clever techniques published for getting the KAPE executable to the target so that an automated, scripted collection can be easily run. **Check out these articles for info on those methods.** Those techniques work exactly as advertised but... they write all the collected data to the target system before forwarding them on to there ultimate destination on an sftp server, S3, Microsoft Azure. As discussed earlier, if there is an intention to do a full forensic image of the target at some point or even the possibility of needing the full image, we really don't want to overwrite unallocated unless there is no other choice. So, how does this work?

All we need to do is:

1. Create a ZeroTier Account. This account can be used for many collections and we can create up to 100 networks with this free account. We will remove the network after the collection.
2. Install ZeroTier on both the collection system and the remote system.
3. Join both the collection system and the remote system to the ZeroTier network we setup for this collection. Note that joining systems to a network requires approval for every machine joined.
4. Approve the nodes for both systems once they join the network. This is done from a web console on the collection system. 
5. Map the drive from the collection system, our destination for the collected files, exactly as we did in the LAN environment.
6. Run the KAPE collection from the Target.

Getting ZeroTier installed on the system is going to require some assistance to get control of the Target system. This is going to be the case with most remote collections so this is not unique to using ZeroTier. If already installed, we could use Zoom or some other software that will allow us to remotely control the system.  Possibly a remote IT person or even the custodian (the person to whom the system belongs) themselves can help with the ZeroTier installation if screen sharing is not available. Once the target is connected to the ZeroTier system, we can use RDP to access the the remote Target over the ZeroTier network. Regardless of the how we accomplish it, we need to get ZeroTier installed and connected to the ZeroTier network. This is a very simple procedure.

Let's walk though this step by step.

### Create a ZeroTier Account

In you web browser navigate to zerotier.com and select "Sign Up".

![zt_home](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_home.png)

Complete the registration page and click "register".

Check your email and activate your new ZeroTier account.

![zt_activate](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_activate.png)



Log into ZeroTier for the first time and click "Create your first ZeroTier Network"

![zt_1st_network](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_1st_network.png)

You will now see your first network. Click the Network ID link to get to the network details page. You will not need to make any changes on this page at this time but please explore if you wish to see the configuration options.

![zt_networks](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_networks.png)



Next we need to download and install the ZeroTier app on both the collection system and the Target. The download links can be found in Several locations on the ZeroTier site. Since we are on the network details page, we can get get the download link here so we can download the ZeroTier client and install in both the collection system and the Target.

![zt_download_link](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_download_link.png)

From the ZeroTier downloads page, download the Windows installer. Once downloaded, double click the installer in your Downloads folder and run the installer accepting all the defaults.

![zt_window_installer_link](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_window_installer_link.png)

![zt_installer](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_installer.png)

Click "Finish" and the ZeroTier client will be launched and minimized to your Windows System Tray.  Right click the ZeroTier Icon in the system tray, select "Join Network", enter the Network ID, and click the "Join" button. The system will now connect to the Network. The system will have to authorized from the ZeroTier system console (where we got the network ID). Remember, perform this download, install and configuration for both the collection system and the target. The collection system could/should be setup ahead of time. Make note of the Node ID so you can label the collection system and the target in the management console.

![zt_join_network](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_join_network.png)

When the system joins the network you will see this Windows notification, select "Yes".

![zt_discoverable](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_discoverable.png)

Now we need to authorize the system on the network via the ZeroTier web-based management console. Login to your ZeroTier account and select **"My Networks"**. Then select the network ID that we have joined the two systems to. Scroll down to the **"Member"** section. In our example, only the two systems will be displayed. Your configuration could have more if you have added more systems to this network.  

Check the boxes to the left of each of the systems to authorize them.

Add a description to each system so you know which is which based upon the Node ID you noted when you configured the client.

Make note of the IPs assigned to each system. You will need these to map the shares and RDP to the systems.

![zt_members_2](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_members_2.png)



At this point both systems are live on the private network. You can verify this by pinging from one to the other, both directions.

![zt_ping](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_ping.png)

Now you need to duplicate the steps that we did above when we shared the the collection system folder containing the kape.exe executable and then on the target system map that share to a drive letter.

![zt_net_use_from_target](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/zt_net_use_from_target.png)

Now we can run the KAPE collection from the Target, executing the kape.exe from the collection system and writing the files directly to the collection system. This is the exact same command that we used above (because we mapped to the same drive letter).

```
k:\kape\kape.exe --tsource C --target RegistryHives --tdest k:\kape_out\tdest --vss
```

![kape_from_target_reg](https://raw.githubusercontent.com/EricZimmerman/KapeDocs/master/External/Remote_Collections_KAPE/kape_from_target_reg.png)



There you have it, remote collections with KAPE over a private network that you created in minutes. Create an ZeroTier account and a couple of VMs and test this out. Enjoy! 

