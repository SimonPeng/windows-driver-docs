---
title: Calling Order for Minidriver Functions
description: Calling Order for Minidriver Functions
ms.date: 04/20/2017
---

# Calling Order for Minidriver Functions





When a minidriver is started, it calls some of the older STI entry points, such as [**IStiUSD::Initialize**](/windows-hardware/drivers/ddi/stiusd/nf-stiusd-istiusd-initialize), and [**IStiUSD::GetStatus**](/windows-hardware/drivers/ddi/stiusd/nf-stiusd-istiusd-getstatus). As soon as the first application attempts to communicate with the device, the WIA service calls [**IWiaMiniDrv::drvInitializeWia**](/windows-hardware/drivers/ddi/wiamindr_lh/nf-wiamindr_lh-iwiaminidrv-drvinitializewia). It is in this function that the minidriver should construct the item tree.

The WIA service next calls [**IWiaMiniDrv::drvInitItemProperties**](/windows-hardware/drivers/ddi/wiamindr_lh/nf-wiamindr_lh-iwiaminidrv-drvinititemproperties) for each item in the tree. The minidriver should create all the properties relevant to the item. In some situations, it might be wise to create an empty property and fill in its data later. For example, for better performance, image thumbnails on cameras should be read in only when the WIA service specifically asks for them, as described below.

The next function to be called depends on the application and device type. Typically, an application's most common operation is transferring data. For scanners, the application first sets up the properties (for example, data type and extent) that define the image it wants to get from the device. The WIA service calls [**IWiaMiniDrv::drvValidateItemProperties**](/windows-hardware/drivers/ddi/wiamindr_lh/nf-wiamindr_lh-iwiaminidrv-drvvalidateitemproperties) when the application changes any properties. The minidriver should check that the properties are valid, communicating with the device, if necessary. The minidriver should generally avoid setting the properties in that function, because another application could set the properties to different values before the data transfer takes place.

To transfer data, the WIA service calls [**IWiaMiniDrv::drvLockWiaDevice**](/windows-hardware/drivers/ddi/wiamindr_lh/nf-wiamindr_lh-iwiaminidrv-drvlockwiadevice), [**IWiaMiniDrv::drvWriteItemProperties**](/windows-hardware/drivers/ddi/wiamindr_lh/nf-wiamindr_lh-iwiaminidrv-drvwriteitemproperties), [**IWiaMiniDrv::drvAcquireItemData**](/windows-hardware/drivers/ddi/wiamindr_lh/nf-wiamindr_lh-iwiaminidrv-drvacquireitemdata), and [**IWiaMiniDrv::drvUnLockWiaDevice**](/windows-hardware/drivers/ddi/wiamindr_lh/nf-wiamindr_lh-iwiaminidrv-drvunlockwiadevice), in that order. The calls to lock and unlock the device guarantee that no other application accesses the device during the transfer. For scanners, **IWiaMiniDrv::drvWriteItemProperties** should send properties such as position, extent, and resolution to the device. Camera drivers typically do not need to send any properties to the device. **IWiaMiniDrv::drvAcquireItemData** should retrieve the image data from the device and send it back to the application via the WIA service, using the [IWiaMiniDrvCallback COM interface](iwiaminidrvcallback-com-interface.md).

For cameras, if an application wants to display thumbnails for the images, the WIA service calls [**IWiaMiniDrv::drvReadItemProperties**](/windows-hardware/drivers/ddi/wiamindr_lh/nf-wiamindr_lh-iwiaminidrv-drvreaditemproperties) on each image. The minidriver should read the thumbnail at that point and cache it in the driver item context. It is important to cache the thumbnail, because multiple applications might ask for the thumbnail, resulting in multiple calls to **IWiaMiniDrv::drvReadItemProperties**. If the minidriver reads the thumbnail each time an application asks for it, performance suffers.

One other special consideration for cameras is root item properties that affect settings on the camera (shutter speed, for example). When the application changes these properties, the WIA service calls [**IWiaMiniDrv::drvValidateItemProperties**](/windows-hardware/drivers/ddi/wiamindr_lh/nf-wiamindr_lh-iwiaminidrv-drvvalidateitemproperties). The minidriver can communicate with the camera, if necessary, to validate the property settings. This function, however, is not the best place to change settings on the camera, because another application also can change the properties. The minidriver should update all of the camera settings from the root item properties when the [**IWiaMiniDrv::drvDeviceCommand**](/windows-hardware/drivers/ddi/wiamindr_lh/nf-wiamindr_lh-iwiaminidrv-drvdevicecommand) function is called to capture a new image.

 

