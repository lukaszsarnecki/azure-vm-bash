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
## Ustawienie zmiennych

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
