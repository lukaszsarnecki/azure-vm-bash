# Jak stworzyć VM w azure za pomocą BASHa
Aby utworzyć maszynę wirtualną (VM) w Microsoft Azure przy użyciu Bash, należy wykonać kilka kroków, zaczynając od zalogowania się do Azure i stworzenia podstawowych zasobów, takich jak grupa zasobów, sieć, podsieć, a na końcu sama maszyna wirtualna. Zakładam, że masz już zainstalowane i skonfigurowane Azure CLI. Oto jak można to zrobić krok po kroku.

#### UWAGA: Przypisane poniżej zmienne pełnią role przykłądu oraz parametry są formą przykładu.

## Krok 1: Zaloguj się do Azure

```bash
az login
```
Jeśli pracujesz na serwerze, gdzie nie ma interfejsu graficznego, użyj:

```bash
az login --use-device-code
```
## Krok 2: Ustawienie zmiennych

Najpierw zdefiniujmy zmienne, które będziemy używać w dalszej części skryptu. Dzięki temu łatwiej będzie zmieniać parametry maszyny.
```bash
RESOURCE_GROUP="myResourceGroup"
LOCATION="westeurope"
VM_NAME="myVM"
VM_IMAGE="UbuntuLTS"   # Możesz zmienić na inny obraz, np. "Win2019Datacenter, listę obrazów można sprawdzić komendą #### az vm image list"
VM_SIZE="Standard_B1s" # Inny rozmiar np. "Standard_DS1_v2"
ADMIN_USERNAME="azureuser"
ADMIN_PASSWORD="TwojeHasło123!" # Używane w przypadku VM Windows lub loginu z hasłem
SSH_KEY_PATH="~/.ssh/id_rsa.pub" # Ścieżka do pliku z kluczem publicznym (dla VM Linux)
```
## Krok 3: Stwórz grupę zasobów

```bash
az group create --name $RESOURCE_GROUP --location $LOCATION
```
## Krok 4: Utwórz sieć wirtualną i podsieć
``` bash
az network vnet create --resource-group $RESOURCE_GROUP --name "${VM_NAME}VNet" --subnet-name "${VM_NAME}Subnet"
```
## Krok 5: Utwórz interfejs sieciowy (NIC)
```bash
az network nic create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name "${VM_NAME}VNet" \
  --subnet "${VM_NAME}Subnet" \
  --name "${VM_NAME}NIC"
```
## Krok 6: Utwórz maszynę wirtualną
### Dla VM z systemem Linux (z kluczem SSH)
Jeśli tworzysz maszynę Linux, użyj klucza SSH:
```bash
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --nics "${VM_NAME}NIC" \
  --image $VM_IMAGE \
  --size $VM_SIZE \
  --admin-username $ADMIN_USERNAME \
  --ssh-key-value $SSH_KEY_PATH
```
### Dla VM z systemem Windows (z hasłem)
Jeśli tworzysz maszynę Windows, użyj hasła:
```bash
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --nics "${VM_NAME}NIC" \
  --image "Win2019Datacenter" \
  --size $VM_SIZE \
  --admin-username $ADMIN_USERNAME \
  --admin-password $ADMIN_PASSWORD
```
## Krok 7: Otwórz porty (np. 22 dla SSH lub 3389 dla RDP)
Aby uzyskać dostęp do maszyny, możesz otworzyć odpowiednie porty:
### Dla VM Linux (port SSH)
```bash
az vm open-port --port 22 --resource-group $RESOURCE_GROUP --name $VM_NAME
```
### Dla VM Windows (port RDP)
```bash
az vm open-port --port 3389 --resource-group $RESOURCE_GROUP --name $VM_NAME
```
## Krok 8: Uzyskaj informacje o maszynie wirtualnej
Na końcu warto sprawdzić publiczny adres IP maszyny:
```bash
az vm list-ip-addresses --resource-group $RESOURCE_GROUP --name $VM_NAME --output table
```

### Przykładowy kompletny skrypt Bash
Oto pełny skrypt, który można uruchomić, aby stworzyć maszynę wirtualną w Azure:
```bash
#!/bin/bash

# Zmienne
RESOURCE_GROUP="myResourceGroup"
LOCATION="westeurope"
VM_NAME="myVM"
VM_IMAGE="UbuntuLTS"
VM_SIZE="Standard_B1s"
ADMIN_USERNAME="azureuser"
SSH_KEY_PATH="~/.ssh/id_rsa.pub"

# Tworzenie grupy zasobów
az group create --name $RESOURCE_GROUP --location $LOCATION

# Tworzenie sieci wirtualnej i podsieci
az network vnet create --resource-group $RESOURCE_GROUP --name "${VM_NAME}VNet" --subnet-name "${VM_NAME}Subnet"

# Tworzenie interfejsu sieciowego
az network nic create --resource-group $RESOURCE_GROUP --vnet-name "${VM_NAME}VNet" --subnet "${VM_NAME}Subnet" --name "${VM_NAME}NIC"

# Tworzenie maszyny wirtualnej
az vm create --resource-group $RESOURCE_GROUP --name $VM_NAME --nics "${VM_NAME}NIC" --image $VM_IMAGE --size $VM_SIZE --admin-username $ADMIN_USERNAME --ssh-key-value $SSH_KEY_PATH

# Otwieranie portu SSH (dla VM z Linux)
az vm open-port --port 22 --resource-group $RESOURCE_GROUP --name $VM_NAME

# Wyświetlenie publicznego adresu IP
az vm list-ip-addresses --resource-group $RESOURCE_GROUP --name $VM_NAME --output table
```
