# Jak stworzyć VM w azure za pomocą BASHa
Aby utworzyć maszynę wirtualną (VM) w Microsoft Azure przy użyciu Bash, należy wykonać kilka kroków, zaczynając od zalogowania się do Azure i stworzenia podstawowych zasobów, takich jak grupa zasobów, sieć, podsieć, a na końcu sama maszyna wirtualna. Zakładam, że masz już zainstalowane i skonfigurowane Azure CLI. Oto jak można to zrobić krok po kroku.

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
VM_IMAGE="UbuntuLTS"   # Możesz zmienić na inny obraz, np. "Win2019Datacenter"
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
