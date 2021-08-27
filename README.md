# VSC Setup

## Notes
For the most part, this guide will assume you are on Windows. I have not gone through this process on any MacOS machine, but I know it is easily done on Linux with very similar steps. If you aren't on Windows, keep this in mind and skip/modify steps as necessary.

## Requirements  
- [MSYS2](https://msys2.org). Follow all installation instructions. Preferred package manager works for Linux machines. Remember where this program is installed - ideally in a filepath that lacks spaces. I recommend simply `C:/msys64/`
- VS Code. Duh.
- [C/C++ extension pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack) for VSC
- [Makefile Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.makefile-tools) extension for VSC
- UTK's version of [Pulse Secure](https://help.utk.edu/kb/index.php?func=show&e=2712). Follow all installation instructions.

## Steps
### Within MSYS
1. Run `pacman -S openssh`
2. Run `ssh-keygen`. It will bring up several prompts, just press enter. Don't type anything.
3. Run `ssh-copy-id netid@tesla`, making necessary substitutions. For example, my NetID is ipelton and I use tesla1. Thus, I would run `ssh-copy-id ipelton@tesla1.eecs.utk.edu`. When prompted for a password, enter your NetID password.
4. (Optional) If you want a compiler to do some stuff locally, run `pacman -S mingw-w64-x86_64-gcc`

### Modifying PATH variables
1. In the Start Menu, search the word "environment" and press enter. Click the button at the bottom of this window that says "Environment Variables..."
2. In the lower half of the window that pops up, scroll until you find your "Path" variable. Select it, and click "Edit..." 

    ![Environment variables example](https://i.imgur.com/AQyW3P1.png)
3. Press "New" on the left bar. Assuming you installed MSYS in `C:/msys64/`, add the following: `C:\msys64\usr\bin`
4. If you did the optional step for a compiler, also add `C:\msys64\mingw64\bin`
5. By default, both of these new additions will appear at the bottom of the list, meaning they have lowest priority. At the very least, we want them to take priority over the default Windows equivalents. Thus, select them and click "Move Up" on the left until they are at the top.
6. Press "OK" on all windows to ensure your changes save.
7. You may test that these steps worked by opening Powershell and running `get-command ssh`. It should look like the following image - note the source on the right.

    ![PATH verification](https://i.imgur.com/rkFrNrj.png)

### Setting up network drive
1. Open up Pulse Secure and connect to your UTK VPN.
2. Open up a File Explorer (winkey + E) and type the following into the filepath bar: `\\fs1.eecs.utk.edu\netid`, filling in your NetID accordingly.

    ![Filepath bar](https://i.imgur.com/IVwY4zf.png)
3. Upon hitting enter, you may be prompted to log in. Sometimes, this prompt won't pop up immediately but rather hide among your open file explorer windows. If it does not load immediately or give you this login prompt, check your open windows.
4. If prompted, log in using `netid@vols.utk.edu` and your NetID password. Check the box to remember your credentials. If you change your NetID password, you will have to navigate in the Control Panel to `Control Panel\User Accounts\Credential Manager` to modify your saved password there.
5. If everything has been done right so far, you should see your Tesla drive.

    ![Tesla drive](https://i.imgur.com/7j6FVCD.png)

6. Back out one folder by clicking on the up arrow to the left of the filepath bar, or clicking on `fs1.eecs.utk.edu`. You should now see three folders, one titled with your NetID.
7. Right-click on the folder with your NetID and select "Map as network drive." You may choose whatever drive letter you like, but keep all other default options.
8. You should now see a new drive on the sidebar in your file explorer under "This PC." As long as you are on a UTK network or using the Pulse Secure VPN, this new drive is accessible and is functionally equivalent to the file sidebar from MobaXTerm, or other FTP clients.

### Configuring SSH
1. Navigate to your MSYS2 folder. From there, go to `/home/$USERNAME$/.ssh`
2. Right click and create a new .txt document.
3. Paste the following into the document:
    ```
    Host alias
        HostName teslacomputer
        User netid
        Compression yes
        ForwardAgent yes
        ForwardX11 yes
        ForwardX11Trusted yes
    ```
4. Make the appropriate substitutions. In this case, the first three lines need to be modified:
    - Replace `alias` with whatever you want to type in the future to SSH into a Tesla machine. For example, if the line reads `Host Tesla`, you can type `ssh Tesla` to connect to a Tesla machine.
    - Replace `teslacomputer` with the address of any Tesla or Hydra machine you want. For example, the line could read `HostName tesla17.eecs.utk.edu`
    - Replace `netid` with your own NetID
5. Save your changes and close the file.
6. Right click on your new text file and rename it to `config`. Note the lack of a file extension - we don't want one anymore. By default, Windows will hide file extensions and make removing them an annoying process. To change this, select "View" at the top of your file explorer and check "File name extensions" on the right.


### Verify everything up to this point
1. Let's make sure everything is in place. Open up a new command prompt. Assuming in the previous steps you chose to follow the example and replaced `alias` with `Tesla`, type `ssh Tesla` into the command prompt. On first attempt, it may ask you about a fingerprint - type yes and hit enter.
2. If everything was done right, it should instantly log into a Tesla machine as you without prompting for a password. If this did work, close the command prompt and try again. If it still does not work, go back and verify you followed all previous steps correctly. Feel free to message me on Discord if things still don't work.

### Setting up VS Code
1. With Pulse Secure connected, you should be able to open up VS Code and select a folder in your Tesla directory through the network drive set up earlier. Select whatever folder within that you want, though I personally suggest simply opening the root. For example, if you mounted the drive at `Z:/`, I should select `Z:/` as the folder to open within VS Code.
2. Since we're sorta on a Tesla machine now, but sorta *not* on a Tesla machine, certain things will act strangely. Most of these won't affect you, but Plank's libraries are one of these cases. Our makefile may know where to look since it truly is running on Tesla, but VS Code is on our machine and has no idea where these files are. To fix this, SSH into the Tesla machine and run `cp -r ~jplank/cs360/include/ ~/cosc360/`
3. In VS Code, press Ctrl+Shift+P and type in "Configurations". Select `C/C++: Edit Configurations (JSON)`
4. Modify the `includePath` variable, adding in the folder we just copied. Since we're working locally, it needs to be the path on your computer, *not* Linux. This is simple.

    ![includePath](https://i.imgur.com/VuWzSPW.png)

    In my case, my network drive is `U:/`, so this is what I had to add to the variable. Simply change it to whatever letter you used and save your changes. You may not make use of these copied files, but what this does is give VS Code a valid path to a copy of these files. This allows Intellisense to detect errors and give predictions on things involving these libraries.
