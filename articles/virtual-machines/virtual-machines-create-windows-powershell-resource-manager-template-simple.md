<properties
	pageTitle="Erstellen eines virtuellen Computers mit einer Vorlage | Microsoft Azure"
	description="Verwenden Sie eine Ressourcen-Manager-Vorlage und Azure PowerShell, um einen neuen virtuellen Windows-Computer zu erstellen."
	services="virtual-machines"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/08/2015"
	ms.author="davidmu"/>

# Erstellen eines virtuellen Windows-Computers mit einer Ressourcen-Manager-Vorlage und PowerShell

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]Klassisches Bereitstellungsmodell.


Sie können ganz einfach einen neuen Windows-basierten virtuellen Computer (VM) erstellen, indem Sie eine Ressourcen-Manager-Vorlage und Azure PowerShell verwenden. Von dieser Vorlage wird ein einzelner virtueller Computer unter Windows in einem neuen virtuellen Netzwerk mit einem einzelnen Subnetz in einer neuen Ressourcengruppe erstellt.

![](./media/virtual-machines-create-windows-powershell-resource-manager-template-simple/windowsvm.png)

[AZURE.INCLUDE [powershell-preview](../../includes/powershell-preview-inline-include.md)]

## Erstellen eines virtuellen Windows-Computers

Gehen folgendermaßen Sie vor, um einen Windows-VM mithilfe einer Ressourcen-Manager-Vorlage im Github-Vorlagenrepository mit Azure PowerShell zu erstellen.

Geben Sie einen Azure-Bereitstellungsnamen, einen Ressourcengruppennamen und den Standort des Azure-Rechenzentrums ein. Führen Sie anschließend die folgenden Befehle aus:

	$deployName="<deployment name>"
	$RGName="<resource group name>"
	$locName="<Azure location, such as West US>"
	$templateURI="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-simple-windows-vm/azuredeploy.json"
	New-AzureRmResourceGroup –Name $RGName –Location $locName
	New-AzureRmResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateUri $templateURI

Beim Ausführen des Befehls **New-AzureRmResourceGroupDeployment** werden Sie aufgefordert, Parameterwerte im Abschnitt „Parameter“ der JSON-Datei anzugeben. Wenn Sie alle Parameterwerte angegeben haben, erstellt der Befehl die Ressourcengruppe und den virtuellen Computer.

	$deployName="TestDeployment"
	$RGName="TestRG"
	$locname="West US"
	$templateURI="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-simple-windows-vm/azuredeploy.json"
	New-AzureRmResourceGroup –Name $RGName –Location $locName
	New-AzureRmResourceGroupDeployment -Name $deployName -ResourceGroupName $RGName -TemplateUri $templateURI

Folgendes sollte angezeigt werden:

	cmdlet New-AzureRmResourceGroupDeployment at command pipeline position 1
	Supply values for the following parameters:
	(Type !? for Help.)
	newStorageAccountName: newsaacct
	adminUsername: WinAdmin1
	adminPassword: *********
	dnsNameForPublicIP: contoso
	VERBOSE: 10:56:59 AM - Template is valid.
	VERBOSE: 10:56:59 AM - Create template deployment 'TestDeployment'.
	VERBOSE: 10:57:08 AM - Resource Microsoft.Network/virtualNetworks 'MyVNET' provisioning status is succeeded
	VERBOSE: 10:57:11 AM - Resource Microsoft.Network/publicIPAddresses 'myPublicIP' provisioning status is running
	VERBOSE: 10:57:11 AM - Resource Microsoft.Storage/storageAccounts 'newsaacct' provisioning status is running
	VERBOSE: 10:57:38 AM - Resource Microsoft.Storage/storageAccounts 'newsaacct' provisioning status is succeeded
	VERBOSE: 10:57:40 AM - Resource Microsoft.Network/publicIPAddresses 'myPublicIP' provisioning status is succeeded
	VERBOSE: 10:57:45 AM - Resource Microsoft.Compute/virtualMachines 'MyWindowsVM' provisioning status is running
	VERBOSE: 10:57:45 AM - Resource Microsoft.Network/networkInterfaces 'myVMNic' provisioning status is succeeded
	VERBOSE: 11:01:59 AM - Resource Microsoft.Compute/virtualMachines 'MyWindowsVM' provisioning status is succeeded


	DeploymentName    : TestDeployment
	ResourceGroupName : TestRG
	ProvisioningState : Succeeded
	Timestamp         : 4/28/2015 6:02:13 PM
	Mode              : Incremental
	TemplateLink      :
	Parameters        :
                    	Name             Type                       Value
	                    ===============  =========================  ==========
	                    newStorageAccountName  String                     newsaacct
	                    adminUsername    String                     WinAdmin1
	                    adminPassword    SecureString
	                    dnsNameForPublicIP  String                     contoso
	                    windowsOSVersion  String                     2012-R2-Datacenter

	Outputs           :

Nun ist ein neuer virtueller Windows-Computer mit dem Namen „MyWindowsVM“ in Ihrer neuen Ressourcengruppe vorhanden.

## Zusätzliche Ressourcen

[Azure Compute-, Netzwerk- und Speicheranbieter unter dem Azure-Ressourcen-Manager](virtual-machines-azurerm-versus-azuresm.md)

[Azure Resource Manager Overview](resource-group-overview.md) (Übersicht über den Azure-Ressourcen-Manager)

[Create a Windows virtual machine with Azure Resource Manager and PowerShell](virtual-machines-create-windows-powershell-resource-manager.md) (Erstellen eines virtuellen Windows-Computers mit dem Azure-Ressourcen-Manager und PowerShell)

[Sie können virtuelle Windows-Computer mit PowerShell und dem klassischen Bereitstellungsmodell erstellen.](virtual-machines-ps-create-preconfigure-windows-vms.md)

[Dokumentation zu virtuellen Computern](http://azure.microsoft.com/documentation/services/virtual-machines/)

[Installieren und Konfigurieren von Azure PowerShell](install-configure-powershell.md)

<!---HONumber=Nov15_HO3-->