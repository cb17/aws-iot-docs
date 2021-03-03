# Device provisioning<a name="iot-provision"></a>

Provisioning credentials for the devices in your fleet is a critical foundational step during the on-boarding process\. Ensuring that your devices have a **strong** unique identity provides a high level of trust within your IoT platform\. The best practice for certificate distribution, like with any credentials management, is to ensure that there's a strong chain of trust in the distribution process\. This means that you can prove who created the certificate and who controlled the certificate all the way to being placed on the device\. 

### What are the risks involved with an insecure provisioning process?
An attacker who can inject themself in the credential provisioning process, can poison the supply chain of devices\. This would call into question the validity of the identity of **all** of your IoT devices and require all devices to be reprovisioned and the provisioning process to be reconfigured\. Furthermore, with the ability to provision devices in AWS IoT, an attacker could start to observe and attack your IoT platform from within the system\.

Fortunately, AWS IoT has a number of services that can provide a simple and **secure** provisioning solution\. 

### MOST Recommended Provisioning Options

**Install certificates on IoT devices before they are delivered**  
Installing a unique set of credentials before devices are delivered to end users can be the most secure option for provisioning\. This is accomplished using our Just-In-Time-Provisioning service\. An example of how this could be implemented is shown below:

![Just-In-Time-Provisioning with ACM Root](https://code.ilovethe.cloud/pythia/jitp_aws_acm_root.png)

Using JITP and JITR, the certificate authority \(CA\) used to sign the device certificate is registered with AWS IoT and is recognized by AWS IoT when the device first connects\. The device is provisioned in AWS IoT on its first connection using the details of its provisioning template\.

For more details on JITP and JITR, see [*just\-in\-time* provisioning \(JITP\)](jit-provisioning.md) or [*just\-in\-time* registration \(JITR\)](auto-register-device-cert.md)\.

An even better scenario is to use a Trusted Platform Module (TPM) to integrate into the JITP/JITR provisioning process. You can read more about TPMs here: https://aws.amazon.com/blogs/iot/using-a-trusted-platform-module-for-endpoint-device-security-in-aws-iot-greengrass/

**End users or installers can use an app to install certificates on their IoT devices**  
If you cannot securely install unique client certificates on your IoT device before they are delivered to the end user, but the end user or an installer can use an app to register the devices and install the unique device certificates, you want to use the [provisioning by trusted user](provision-wo-cert.md#trusted-user) process\.

  Using a trusted user, such as an end user or an installer with a known account, can simplify the device manufacturing process\. Instead of a unique client certificate, devices have a temporary certificate that enables the device to connect to AWS IoT for only 5 minutes\. During that 5\-minute window, the trusted user obtains a unique client certificate with a longer life and installs it on the device\. The limited life of the claim certificate minimizes the risk of a compromised certificate\.

  For more information, see [Provisioning by trusted user](provision-wo-cert.md#trusted-user)\.

### If the above recommended solutions don't work
In some rare cases, we find customers that have use cases that don't fit with the above recommended solutions\. For those customers, they might need to use custom authentication mechanisms to authenticate the devices and return back a certificate. An example of this might be if you're looking to migrate devices from a legacy IoT platform to AWS IoT that use an authentication protocol unsupported by AWS IoT. In these scenarios, you can use [Fleet Provisioning by claim](provision-wo-cert.md#claim-based) with a custom authorizer Lambda function. This provisioning process can be tricky to get right securely. So if you think this is the **only** provisioning process that will work best for you, we encourage you to consult with an AWS IoT expert or your AWS account team for further guidance.

## Provisioning devices in AWS IoT<a name="provisioning-in-iot"></a>

When you provision a device with AWS IoT, you must create resources so your devices and AWS IoT can communicate securely\. Other resources can be created to help you manage your device fleet\. The following resources can be created during the provisioning process: 
+ An IoT thing\.

  IoT things are entries in the AWS IoT device registry\. Each thing has a unique name and set of attributes, and is associated with a physical device\. Things can be defined using a thing type or grouped into thing groups\. For more information, see [Managing devices with AWS IoT](iot-thing-management.md)\.

   Although not required, creating a thing makes it possible to manage your device fleet more effectively by searching for devices by thing type, thing group, and thing attributes\. For more information, see [Fleet indexing service](iot-indexing.md)\.
+ An X\.509 certificate\.

  Devices use X\.509 certificates to perform mutual authentication with AWS IoT\. You can register an existing certificate or have AWS IoT generate and register a new certificate for you\. You associate a certificate with a device by attaching it to the thing that represents the device\. You must also copy the certificate and associated private key onto the device\. Devices present the certificate when connecting to AWS IoT\. For more information, see [Authentication](authentication.md)\.
+ An IoT policy\.

  IoT policies define the operations a device can perform in AWS IoT\. IoT policies are attached to device certificates\. When a device presents the certificate to AWS IoT, it is granted the permissions specified in the policy\. For more information, see [Authorization](iot-authorization.md)\. Each device needs a certificate to communicate with AWS IoT\.

AWS IoT supports automated fleet provisioning using provisioning templates\. Provisioning templates describe the resources AWS IoT requires to provision your device\. Templates contain variables that enable you to use one template to provision multiple devices\. When you provision a device, you specify values for the variables specific to the device using a dictionary or *map*\. To provision another device, specify new values in the dictionary\.

You can use automated provisioning whether or not your devices have unique certificates \(and their associated private key\)\.

