#self-service-macOS-upgrades

We decided early on that we wanted to utilize Self Service for more intrusive tasks, such as macOS upgrades, while still maintaining the flexibility that our existing software deployment approach afforded us (see [automated-software-deployment](https://github.com/ToplessBanana/tutorials/tree/master/HOW-TO-automated-software-deployment)).

##Packages

Since macOS is distributed through the App Store, we need to capture the package while it's being downloaded to a machine. Begin downloading `Install macOS Sierra.app`, but pause the download part way through.

Locate the installer package in the `/private/var/folders/` directory, then use the following `ln` command in Terminal to create a hard link to a file on your desktop called `Install macOS Sierra.pkg`.

`ln /private/var/folders/.../.../com.apple.appstore/.../...pkg Desktop/Install\ macOS\ Sierra.pkg`

Then, resume the download in the App Store and allow it to complete to finish capturing the package.

##Scripts

###postinstall_Install_macOS_Sierra.sh

```bash

#!/bin/sh

mkdir /Applications/Install\ macOS\ Sierra.app/Contents/_MASReceipt/

touch /Applications/Install\ macOS\ Sierra.app/Contents/_MASReceipt/receipt

```

###remove_Install_macOS_Sierra.sh

```bash

#!/bin/sh
#
# remove_Install_macOS_Sierra.sh
# 
#
# Created by Jayson Kish on 12/20/16.
# Copyright 2016 by Topless Banana. All rights reserved.
#
# Version 12.2.03
#
#
#


sudo rm -rf /Applications/Install\ macOS\ Sierra.app
sudo rm /private/var/db/receipts/com.apple.pkg.InstallOS.plist
sudo rm /private/var/db/receipts/com.apple.pkg.InstallOS.bom

#

/bin/rm $0 remove_Install_macOS_Sierra.sh

#

```

##Static Computer Group

Since we typically take a phased approach when rolling out OS upgrades in our environment, we'll need to create a Static Computer Group that a users workstation can be assigned to in order to deploy the installer.

###macOS Sierra

Create a Static Computer Group called `macOS Sierra` now.

##Smart Computer Groups

###deploy_macOS Sierra

Create a Smart Computer Group called `deploy_macOS Sierra` with the following criteria now.

![deploy-macos-sierra-criteria](https://github.com/ToplessBanana/tutorials/blob/master/HOW-TO-self-service-macOS-upgrades/resources/deploy-macos-sierra-criteria.png)

###remove_macOS Sierra

Create a Smart Computer Group called `remove_macOS Sierra` with the following criteria now.

![remove-macos-sierra-criteria](https://github.com/ToplessBanana/tutorials/blob/master/HOW-TO-self-service-macOS-upgrades/resources/remove-macos-sierra-criteria.png)

###exclude_macOS Sierra

Create a Smart Computer Group called `exclude_macOS Sierra` with the following criteria now.

![remove-macos-sierra-criteria](https://github.com/ToplessBanana/tutorials/blob/master/HOW-TO-self-service-macOS-upgrades/resources/exclude-macos-sierra-criteria.png)

###deploy_macOS Sierra (Self Service)

Create a Smart Computer Group called `deploy_macOS Sierra (Self Service)` with the following criteria now.

![deploy-macos-sierra-self-service-criteria](https://github.com/ToplessBanana/tutorials/blob/master/HOW-TO-self-service-macOS-upgrades/resources/deploy-macos-sierra-self-service-criteria.png)

##Policies

###Install_macOS Sierra

Create a Policy called `Install_macOS Sierra` with the following configuration now.

- **General**
  - [x] Recurring Check-In
  - [x] Execution Frequency: Ongoing
- **Package**
  - [x] Packages: `Install macOS Sierra.pkg`
  - [x] Action: Install
- **Scripts**
  - [x] Scripts: `postinstall_Install_macOS_Sierra.sh`
  - [x] Priority: After
- **Maintenance**
  - [x] Update Inventory
- **Scope**
  - [x] Targets: Specific Computers: `deploy_macOS Sierra`
  - [x] Exclusions: `exclude_macOS Sierra`

###Remove_macOS Sierra

Create a Policy called `Remove_macOS Sierra` with the following configuration now.

- **General**
  - [x] Recurring Check-In
  - [x] Execution Frequency: Ongoing
- **Package**
  - [x] Packages: `Install macOS Sierra.pkg`
  - [x] Action: Uninstall
- **Scripts**
  - [x] Scripts: `remove_Install_macOS_Sierra.sh`
  - [x] Priority: After
- **Maintenance**
  - [x] Update Inventory
- **Scope**
  - [x] Targets: Specific Computers: `remove_macOS Sierra`
  - [x] Exclusions: `exclude_macOS Sierra`
  
###macOS Sierra

Create a Policy called `macOS Sierra` with the following configuration now.

- **General**
  - [x] Execution Frequency: Ongoing
- **Files and Processes**
  - [x] Search for Process: `Self Service`: Kill process if found
  - [x] Execute Command: `/Applications/Install\ macOS\ Sierra.app/Contents/Resources/startosinstall --applicationpath /Applications/Install\ macOS\ Sierra.app/ --volume / --nointeraction`
- **Scope**
  - [x] Targets: Specific Computers: `deploy_macOS Sierra (Self Service)`
  - [x] Exclusions: `exclude_macOS Sierra`
- **Self Service**
  - [x] Make the policy available in Self Service
  - [x] Button Name: `Upgrade`
  - [x] Description: `Update your workstation to macOS Sierra. We recommend that you begin just prior to leaving for the day.`
  - [x] Ensure that users view the description

##Putting It All Together

![macos-sierra-self-service.png](https://github.com/ToplessBanana/tutorials/blob/master/HOW-TO-self-service-macOS-upgrades/resources/macos-sierra-self-service.png)