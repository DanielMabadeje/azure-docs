---
title: Migrate your block data to Azure with Cirrus Data
titleSuffix: Azure Storage
description: Provides quick start guide to implement Cirrus Migrate Cloud, and migrate your data to Azure 
author: dukicn
ms.author: nikoduki
ms.date: 09/06/2021
ms.topic: conceptual
ms.service: storage
ms.subservice: partner
---

# Migrate your block data to Azure with Cirrus Migrate Cloud

Cirrus Migrate Cloud (CMC) enables disk migration from an existing storage system, or cloud to Azure. Migration is performed while the original system is still in operation. This document will present the methodology to successfully configure and execute the migration.

## Overview

The solution uses distributed Migration Agents running on every host that allows direct Host-to-Host connections. Each Host-to-Host migration is independent making the solution infinitely scalable, without central bottlenecks for the dataflow. The migration is using cMotion™ technology to ensure no impact on production. 

## Use cases

This document covers a generic migration case for moving the application from one virtual machine (running on-premises or in another cloud provider) to a virtual machine in Azure. For deeper step-by-step guides in various use cases, you can learn more on the following links:

- [Moving the workload to Azure with cMotion](https://support.cirrusdata.cloud/en/article/howto-cirrus-migrate-cloud-on-premises-to-azure-1xo3nuf/)
- [Moving from Premium Disks to Ultra Disks](https://support.cirrusdata.cloud/en/article/howto-cirrus-migrate-cloud-migration-between-azure-tiers-sxhppt/)
- [Moving from AWS to Azure](https://support.cirrusdata.cloud/en/article/howto-cirrus-migrate-cloud-migration-from-aws-to-azure-weegd9/.)

## Components

Cirrus Migrate Cloud consists of multiple components:

- **cMotion™** feature of CMC does a storage-level cut-over from a source to the target cloud without downtime to the source host. cMotion™ is used to swing the workload over from the original FC or iSCSI source disk to the new destination Azure Managed Disk.
- **Web-based Management Portal** is web-based management as a service. It allows users to manage migration and protect any block storage. Web-based Management Portal provides interfaces for all CMC application configurations, management, and administrative tasks.

    :::image type="content" source="./media/cirrus-data-migration-guide/cirrus-web-portal.jpg" alt-text="Screenshot of CMC Portal":::

## Implementation guide

User should follow the Azure best practices to implement a new virtual machine. If not familiar with the process, learn more from [quick start guide](/azure/virtual-machines/windows/quick-create-portal).

Before starting the migration, make sure the following prerequisites have been met:

- Verify that the OS in Azure is properly licensed
- Verify access to the Azure Virtual Machine
- Check that the application / database license is available to run in Azure
- Check the permission to auto-allocate the destination disk size
- Ensure that managed disk is the same size or larger than the source disk 
- Ensure that either the source, or the destination virtual machine has a port open to allow our H2H connection.

1. **Prepare the Azure virtual machine**. Document is assuming that virtual machine is fully implemented. So, once the data disks are migrated, the destination host can immediately start up the application, and bring it online. State of the data will be the same as the source when it was shut down seconds ago. CMC does not migrate the OS disk from source to destination.

1. **Prepare the application in the Azure virtual machine**. In this example, the source is Linux host. It can run any user application accessing the respective BSD storage. We will use a database application running at the source using a 1 GiB disk as a source storage device. However, any application can be used instead. Set up a virtual machine in Azure ready to be used as the destination virtual machine. Make sure that resource configuration and operating system are compatible with the application, and ready to receive the migration from the source using CMC portal. The destination block storage device/s will be automatically allocated and created during the migration process.

1. **Sign up for CMC account**. To obtain a CMC account, follow the support page for the exact instructions on how to get an account. More details can be read [here](https://support.cirrusdata.cloud/en/article/licensing-m4lhll/).

1. **Create a Migration Project** reflecting the specific migration characteristics, type, owner of the migration, and any details needed to define the operations. 

    :::image type="content" source="./media/cirrus-data-migration-guide/cirrus-create-project.jpg" alt-text="Screenshot for creating a new project":::

1. **Define the migration project parameters**. Use the CMC web-based portal to configure the migration by defining the parameters: source, destination, and other parameters.

1. **Install the migration CMC agents on source and destination hosts**. Using the CMC web-based management portal, select  **Deploy Cirrus Migrate Cloud** to get the curl command for **New Installation**. Run the command on the source and destination command-line interface.

1. **Create a bidirectional connection between source and destination hosts**. Use **H2H** tab in the CMC web-based management portal, and **Create New Connection** button. Select the device used by the application, not the device used by the Linux operating system.

    :::image type="content" source="./media/cirrus-data-migration-guide/cirrus-migration-1.jpg" alt-text="Screenshot that shows list of deployed hosts":::

    :::image type="content" source="./media/cirrus-data-migration-guide/cirrus-migration-2.jpg" alt-text="Screenshot that shows list of host-to-host connections":::

    :::image type="content" source="./media/cirrus-data-migration-guide/cirrus-migration-3.jpg" alt-text="Screenshot that shows list of migrated devices":::

1. **Start the migration to the destination virtual machine** using **Migrate Host Volumes** from the CMC web-based management portal. Follow the instructions for remote location. Use the CMC portal to **Auto allocate destination volumes** on the right of the screen. 
 
1. Next, we need to add Azure Credentials to allow connectivity and disk provisioning using the **Integrations** tab on the CMC portal. Fill in the required fields using your private company’s values for Azure: **Integration Name**, **Tenant ID**, **Client/Application ID**, and **Secret**. Press **Save**. 

    :::image type="content" source="./media/cirrus-data-migration-guide/cirrus-migration-4.jpg" alt-text="Screenshot that shows entering Azure credentials":::

    For details on creating Azure AD application, view our [step-by-step instructions](https://support.cirrusdata.cloud/en/article/creating-an-azure-service-account-for-cirrus-data-cloud-tw2c9n/). By creating and registering Azure AD application for CMC, you enable automatic creation of Azure Managed Disks on the target virtual machine.

    >[!NOTE]
    >Since you selected **Auto allocate destination volumes** on the previous step, don't press it again for a new allocation. If you do, it will output and error. Instead press **Continue**.

## Migration guide

After pressing **Save** in the previous step, **New Migration Session** window appears. Fill in the fields:
   - **Session description**: provide meaningful description
   - **Auto Resync Interval**: enable migration schedule 
   - Use iQoS to select the impact migration will have on the production:
     - **Minimum** throttles migration rate to 25% of the available bandwidth
     - **Moderate** throttles migration rate to 50% of the available bandwidth
     - **Aggressive** throttles migration rate to 75% of the available bandwidth
     - **Relentless** doesn't throttle the migration.

       :::image type="content" source="./media/cirrus-data-migration-guide/cirrus-iqos.jpg" alt-text="Screenshot that shows options for iQoS settings":::

Press **Create Session** to start the migration.

From the start of the migration initial sync until cMotion starts, there is no need for a user interaction with CMC.  Only exception is monitoring the progress. You can monitor current status, session volumes and track the changes using the dashboard. 

:::image type="content" source="./media/cirrus-data-migration-guide/cirrus-monitor-1.jpg" alt-text="Screenshot that shows monitoring progress":::

During the migration you can observe the blocks changed on the source device by pressing the Changed Data Map.  

:::image type="content" source="./media/cirrus-data-migration-guide/cirrus-monitor-2.jpg" alt-text="Screenshot that shows changed data map":::

Details on iQoS will show synchronized blocks, and migration status. It also shows that there is no impact to production IO.

:::image type="content" source="./media/cirrus-data-migration-guide/cirrus-monitor-3.jpg" alt-text="Screenshot that shows iQoS details":::

## Moving the workload to Azure with cMotion

After the initial synchronization finishes, we will prepare to move the workload from the source disk to the destination Azure Managed Disk using cMotion™.

### Start cMotion™

At this point, the systems are ready for cMotion™ migration cut-over. 

1. In the CMS portal select **Trigger cMotion™** using Session to switch the workload from the source to the destination disk. To check if the process was done, you can use iostat, or equivalent command. Go to the terminal in the Azure virtual machine, and run *iostat /dev/<device_name>* (for example /dev/sdc), and observe that the IOs are written by the application on the destination disk in Azure cloud.

:::image type="content" source="./media/cirrus-data-migration-guide/cirrus-monitor-4.jpg" alt-text="Screenshot that shows current monitoring status":::

In this state, the workload can be swung, or moved back to the source disk at any time. If you want to revert the production virtual machine, use the **Session Actions** button, and select the **Revert cMotion™** option. We can swing back, and forth as many times we want while the application is running at source host/VM.

When the final cut-over to the destination virtual machine is required, follow the steps:
1. Select **Session Actions**
2. Click the **Finalize Cutover** option to "lock-in" the cut-over to the new Azure virtual machine, and disable the option for source disk to be removed. Stop any other application running in the source host for final host cut-over. 

### Move the application to the destination virtual machine

Once the cut-over has been done, application needs to be switched over to the new virtual machine. To do that, perform the following steps:

1. Stop the application
2. Unmount the migrated device
3. Mount the new migrated device in Azure virtual machine. 
4. Start the same application in Azure virtual machine on the new migrated disk. 
 
Observe that there are no IOs going to source hosts devices by running the iostat command in the source host. Running iostat in Azure virtual machine will show that IO is executing on the Azure virtual machine terminal.

### Complete the migration session in CMC GUI 

The migration step completed when all the IOs were redirected to the destination devices after triggering cMotion™. You can now close the session using **Session Actions**. Click on **Delete Session** to close the migration session. 
As a last step, you will remove the **Cirrus Migrate Cloud Agents** from both source host and Azure virtual machine. To perform uninstall, get the **Uninstall curl command** from **Deploy Cirrus Migrate Cloud** button. Option is in the **Hosts** section of the portal. 

After the agents are removed, migration is fully completed. Now the source application is running in production on the destination Azure virtual machine with locally mounted disks.	

## Support

### How to open a case with Azure

In the [Azure portal](https://portal.azure.com) search for support in the search bar at the top. Select **Help + support** -> **New Support Request**.

### Engaging Cirrus Support

In the CMC portal, select **Help Center** tab on the CMC portal to contact Cirrus Data Solutions support, or go to [CDSI website](https://support.cirrusdata.cloud/en/), and file a support request.

## Next steps
- Learn more on [Azure virtual machines](/azure/virtual-machines/windows/overview)
- Learn more on [Azure Managed Disks](/azure/virtual-machines/managed-disks-overview)
- Learn more on [storage migration](/azure/storage/common/storage-migration-overview)
- [Cirrus Data website](https://www.cirrusdata.com/)
- Step-by-step guides for [cMotion](https://support.cirrusdata.cloud/en/category/howtos-1un623w/)
