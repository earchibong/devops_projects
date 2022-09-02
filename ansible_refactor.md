# How to deploy and configure UAT Web Servers using Ansible imports and roles.

<br>

![project12_architecture](https://user-images.githubusercontent.com/92983658/188095299-218cd955-2dda-4436-aac2-083e095f90a3.png)

<br>

## Code Refactoring:

### Jenkins Job Enhancement

- Go to the `Jenkins-Ansible` server and create a new directory called `ansible-config-artifact`:
`sudo mkdir /home/ubuntu/ansible-config-artifact`

- Change permissions to this directory, so Jenkins could save files there: `sudo chmod -R 0777 /home/ubuntu/ansible-config-artifact`
- In `Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab`
  - search for `Copy Artifact` and `install without restarting Jenkins`

<br>

![copy_artifacts](https://user-images.githubusercontent.com/92983658/188102085-8a39e8e7-2bde-40cb-b85f-71e839761669.png)

<br>

- Create a new Freestyle project and name it `save_artifacts`.
- Configure `save artifacts` to be triggered by completion of the existing `ansible` project.
  - click `configure` under the the project dashboard
  - under `general` tab, click `discard old builds`
   - under `max # of builds to keep` : `2`
  - under `source code management` tab: click none
  - under `build triggers`: click `build after other projects are built`
   -  enter `ansible` for `projects to watch`
   -  `trigger only if build is stable`    

<br>

![general_configure](https://user-images.githubusercontent.com/92983658/188104450-09dd09ff-906b-4f45-be79-e4f31aef535d.png)

<br>

![source_build_configure](https://user-images.githubusercontent.com/92983658/188104929-0da2a52d-ab4d-40ef-8701-2fa6dfcdeb8b.png)

<br>

- create a `Build` step and choose `Copy artifacts from other project` :
  - specify `ansible` as a source project
  - `**` for artifacts to copy
  - `/home/ubuntu/ansible-config-artifact` as a target directory.

<br>

![build_step](https://user-images.githubusercontent.com/92983658/188105979-2dd9a0d7-d78a-48bd-979e-c3a657398605.png)

<br>

