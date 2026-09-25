# DHCP Server version check

This repo holds one file, `version.json`, which CTI's DHCP Server reads at start-up (and from its *Check for updates...* menu item) to learn whether a newer version exists. The program itself, its source and its setup live in Azure DevOps (CTI-AV / Skunkworks / DHCP-Server); `url` is where the program's Download button sends people. Update `version` only after the new setup is merged to `main` there. The rules for the file are in that repo's README, under *Releasing a version*.
