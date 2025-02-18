---
title: HFP Device Startup
description: The HFP Device startup topic discusses what happens when a Bluetooth hands-free profile (HFP) device arrives in the audio system.
ms.date: 04/20/2017
---

# HFP Device Startup


The HFP Device startup topic discusses what happens when a Bluetooth hands-free profile (HFP) device arrives in the audio system.

For each paired HFP device that arrives in the audio system, the Windows HFP driver registers a device interface in the GUID\_DEVINTERFACE\_BLUETOOTH\_HFP\_SCO\_HCIBYPASS class. The audio driver uses device interface notifications to stay informed of all instances of the GUID\_DEVINTERFACE\_BLUETOOTH\_HFP\_SCO\_HCIBYPASS interfaces. The audio driver calls IoRegisterPlugPlayNotification from within its AVStrMiniDevicePostStart driver routine (or from an equivalent Portcls routine) to register a callback to discover the currently installed HFP devices, and to be notified of new HFP devices.

When the audio driver calls IoRegisterPlugPlayNotification, the call is made using the following parameters.

-   EventCategory is set to EventCategoryDeviceInterfaceChange.

-   EventCategoryFlags is typically set to PNPNOTIFY\_DEVICE\_INTERFACE\_INCLUDE\_EXISTING\_INTERFACES in order to receive immediate notifications of existing interfaces. However some alternate audio driver designs might find existing interfaces through other means.

-   EventCategoryData is set to GUID\_DEVINTERFACE\_BLUETOOTH\_HFP\_SCO\_HCIBYPASS.

-   DriverObject is set to the audio driver’s DriverObject.

-   CallbackRoutine is set to a routine in the audio driver that will receive the notifications.

The following sections outline the tasks that the audio driver can performs for each registered instance of a paired HFP device.

## <span id="Handling_interface_instances"></span><span id="handling_interface_instances"></span><span id="HANDLING_INTERFACE_INSTANCES"></span>Handling interface instances


For each interface instance that is registered in the GUID\_DEVINTERFACE\_BLUETOOTH\_HFP\_SCO\_HCIBYPASS class, the audio driver must use the following protocol for communication:

When Windows calls the audio driver’s callback routine that was registered when the audio driver called IoRegisterPlugPlayNotification, Windows passes a symbolic link for the HFP interface, using DEVICE\_INTERFACE\_CHANGE\_NOTIFICATION.*SymbolicLinkName*.

When the audio driver calls IoGetDeviceObjectPointer, the driver uses the symbolic link to get the HFP FileObject and the DeviceObject for the HFP device.

When the audio driver sends IOCTLs to the HFP driver, the driver uses the HFP FileObject and the DeviceObject for the HFP device.

## <span id="Retrieving_static_information"></span><span id="retrieving_static_information"></span><span id="RETRIEVING_STATIC_INFORMATION"></span>Retrieving static information


The audio driver can retrieve static information from the HFP driver. For example, the HFP driver can provide the ksnodetype, the container id and the friendly name of the paired HFP device. The audio driver can use this information to create and initialize a KS filter or filters representing the paired HFP device. The audio driver uses [**IOCTL\_BTHHFP\_DEVICE\_GET\_DESCRIPTOR**](/windows-hardware/drivers/ddi/bthhfpddi/ni-bthhfpddi-ioctl_bthhfp_device_get_descriptor) to get this information.

The audio driver can also retrieve the Bluetooth address of the paired HFP device. Each paired HFP device has a unique Bluetooth address, and this can be useful as a unique identifier string. For more information, see [Obtaining Bluetooth Address of HF Device](obtaining-bluetooth-address-of-hf-device.md).

## <span id="Creating__initializing_audio-specific_filter_factory_context"></span><span id="creating__initializing_audio-specific_filter_factory_context"></span><span id="CREATING__INITIALIZING_AUDIO-SPECIFIC_FILTER_FACTORY_CONTEXT"></span>Creating, initializing audio-specific filter factory context


To create and initialize an audio-specific filter factory context, the audio driver must store the HFP DeviceObject and the HFP FileObject in the filter factory context, and then initialize the IsConnected field to false.

## <span id="Creating_the_KS_filter_factory"></span><span id="creating_the_ks_filter_factory"></span><span id="CREATING_THE_KS_FILTER_FACTORY"></span>Creating the KS filter factory


For each device instance in the GUID\_DEVINTERFACE\_BLUETOOTH\_HFP\_SCO\_HCIBYPASS interface class, the audio driver creates and enables one or more filter factories.

If the audio driver is an AVStream driver, the audio driver calls KsCreateFilterFactory to add the new filter factory and KsFilterFactorySetDeviceClassesState to enable the factory. If the audio driver is a PortCls driver, then it indirectly creates and enables KS filter factories by calling PcRegisterSubdevice. For many PortCls audio driver designs, there are two sub-devices registered for a given paired HFP device.

Each filter factory (or, for PortCls audio drivers, each pair of filter factories) represents the audio functionality of a single paired HFP device. The audio driver creates separate filter factories for each paired HFP device represented by unique instances of GUID\_DEVINTERFACE\_BLUETOOTH\_HFP\_SCO\_HCIBYPASS interfaces. For each paired HFP device, the audio driver must use unique strings for the *RefString* parameter of KsCreateFilterFactory, or the *Name* parameter of PcRegisterSubdevice. The driver developer might find it useful to use the paired HFP device’s Bluetooth address string as a unique string. See [Obtaining Bluetooth Address of HF Device](obtaining-bluetooth-address-of-hf-device.md) for information about how to retrieve the unique string.

Note that there is no specific maximum number of possible paired HFP devices, so the audio driver should avoid hard coding specific limitations. Instead, the audio driver must correctly handle dynamic arrival and removal of multiple GUID\_DEVINTERFACE\_BLUETOOTH\_HFP\_SCO\_HCIBYPASS interfaces.

As a practical matter, however, a PortCls driver must specify a maximum number of sub-devices when it calls PcAddAdapterDevice. PcAddAdapterDevice pre-allocates extra memory for each potential sub-device. The audio driver developer should select a number high enough to accommodate many paired devices, but at the same time select a number that doesn't result in a waste of resources. For example, supporting only 2 HFP devices might be inadequate, and supporting 2000 would certainly result in overextended resources. However, supporting 16 is likely to be reasonable.

If at runtime the audio driver is notified of another GUID\_DEVINTERFACE\_BLUETOOTH\_HFP\_SCO\_HCIBYPASS interface, but has already registered its maximum number of sub-devices, then the audio driver can invoke some algorithm to choose a paired HFP device whose sub-devices it can unregister to make room for the new HFP device. For example, the audio driver could keep track of the HFP device with the oldest connection. Whereas a simpler but perhaps less user-friendly audio driver could simply ignore additional GUID\_DEVINTERFACE\_BLUETOOTH\_HFP\_SCO\_HCIBYPASS interface after reaching its maximum.

## <span id="Sending_the_get_connection_status_IOCTL"></span><span id="sending_the_get_connection_status_ioctl"></span><span id="SENDING_THE_GET_CONNECTION_STATUS_IOCTL"></span>Sending the get connection status IOCTL


The audio driver sends the get connection status IOCTL to get information about any changes that have occurred in the connection.

## <span id="Sending_the_get_volume_status_IOCTL"></span><span id="sending_the_get_volume_status_ioctl"></span><span id="SENDING_THE_GET_VOLUME_STATUS_IOCTL"></span>Sending the get volume status IOCTL


The audio driver sends the get volume status IOCTL to get information about any changes in volume level that have occurred in the volume status of the headset.

## <span id="related_topics"></span>Related topics
[**IOCTL\_BTHHFP\_DEVICE\_GET\_DESCRIPTOR**](/windows-hardware/drivers/ddi/bthhfpddi/ni-bthhfpddi-ioctl_bthhfp_device_get_descriptor)  
[Theory of Operation](theory-of-operation.md)  
[Obtaining Bluetooth Address of HF Device](obtaining-bluetooth-address-of-hf-device.md)
