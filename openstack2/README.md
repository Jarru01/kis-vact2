# OpenStack Heat – Prehľad úloh (task1–task10)

---

## Úloha 1 (`task1.yml`)

**Zadanie:** Router s jednou internou sieťou (`192.168.88.0/24`) a dvoma VM. PC1 má povolenú ICMP + DNS, PC2 má povolenú ICMP + HTTP/S.

```
Internet (ext-net-154)
        |
    [Router]
        |
  [192.168.88.0/24]
     /        \
  [PC1]      [PC2]
 ICMP+DNS   ICMP+HTTP/S
```

---

## Úloha 2 (`task2.yml`)

**Zadanie:** Jeden router prepájajúci dve interné siete. PC1 v sieti `.33.0/24`, PC2 v sieti `.35.0/24`. Povolený ICMP, FTP na oba, HTTP/S iba pre PC2, komunikácia medzi sieťami.

```
Internet (ext-net-154)
        |
    [Router]
     /      \
[net-33]   [net-35]
192.168.33.0/24   192.168.35.0/24
   |                   |
 [PC1]               [PC2]
ICMP+FTP         ICMP+FTP+HTTP/S
```

---

## Úloha 3 (`task3.yml`)

**Zadanie:** Jeden router, dve siete. Sieť A (`10.0.1.0/24`) s PC1, Sieť B (`172.16.99.0/24`) s PC2 a PC3. ICMP + SSH + RDP všade, HTTP/S + DNS len z UNIZA (`158.193.0.0/16`), komunikácia medzi sieťami.

```
Internet (ext-net-154)
        |
    [Router]
     /         \
[10.0.1.0/24]  [172.16.99.0/24]
    |               /      \
  [PC1]          [PC2]    [PC3]
        všetky: ICMP, SSH, RDP
        HTTP/S+DNS: len z 158.193.0.0/16
```

---

## Úloha 4 (`task4.yml`)

**Zadanie:** Router-VM (dual-homed – externá + interná sieť) s NAT/IP forward. Za ním interná sieť `10.255.55.0/24` s PC1 a PC2. SSH+RDP len z UNIZA, HTTP/S+DNS len z RFC 1918, PC2 má navyše SMB.

```
Internet (ext-net-154)
        |
  [Router-VM]  <-- dual-homed (ext + int), NAT/ip_forward
        |
  [10.255.55.0/24]
     /              \
  [PC1]           [PC2]
  ICMP             ICMP
  SSH+RDP (UNIZA)  SSH+RDP (UNIZA)
  HTTP/S+DNS (RFC1918)  HTTP/S+DNS (RFC1918)
                   + SMB (445, 139, 137-138)
```

---

## Úloha 5 (`task5.yml`)

**Zadanie:** Dve VM pripojené priamo do externej siete (bez routera/internej siete). Povolený ICMP, šifrovaný mail (SMTPS 465, STARTTLS 587, IMAPS 993, POP3S 995), SSH iba pre PC1, RDP iba pre PC2, vzájomná komunikácia cez `remote_group_id`.

```
Internet (ext-net-154)
     /          \
  [PC1]        [PC2]
  SSH           RDP
  ICMP, TLS mail (465,587,993,995)
  <----komunikácia medzi PC---->
```

---

## Úloha 6 (`task6.yml`)

**Zadanie:** Jump server (dual-homed), Router s internou sieťou `10.0.0.0/24`, Application server (Apache2, fixná IP `10.0.0.25`) a Proxy server (Squid3, fixná IP `10.0.0.10`). Inštalácia služieb cez `user_data`.

```
         Internet (ext-net-154)
          /              \
  [Router]             [Jump-VM]
158.193.153.x/24    158.193.153.y/24 (ext)
  10.0.0.1/24         10.0.0.x/24   (int)
       |                    |
       +----[10.0.0.0/24]---+
               /        \
    [App:10.0.0.25]  [Proxy:10.0.0.10]
       Apache2              Squid3
```

---

## Úloha 7 (`task7.yml`)

**Zadanie:** VM v externej sieti + automatické vygenerovanie SSH key-pair priamo v šablóne (`OS::Nova::KeyPair`, `save_private_key: true`). Privátny kľúč sa vypíše cez `outputs`.

```
Internet (ext-net-154)
        |
      [VM]  <-- key: kluc-uloha7
        
Outputs:
  private_key --> PEM kľúč pre SSH prístup
```

---

## Úloha 8 (`task8.yml`)

**Zadanie:** VM v externej sieti s dynamickým menom podľa názvu stacku (`{stack-name}-server`). IPv4 adresa VM sa vypíše cez `outputs`.

```
Internet (ext-net-154)
        |
  [{stack-name}-server]

Outputs:
  vm_ipv4_address --> pridelená IP adresa
```

---

## Úloha 9 (`task9.yml`)

**Zadanie:** VM s Security Group (SSH port 22 vždy povolený). Telnet (port 23) sa povolí iba ak používateľ nastaví parameter `povolenie_telnetu: true` (predvolene `false`). Použitie `conditions`.

```
Internet (ext-net-154)
        |
  [server-uloha9]
        |
   [sg-uloha9]
    TCP 22 (vždy)
    TCP 23 (len ak povolenie_telnetu=true)
```

---

## Úloha 10 (`task10.yml`)

**Zadanie:** Dynamický počet VM podľa vstupu používateľa (parameter `pocet_vm`, default 2, max 5). Použitie `OS::Heat::ResourceGroup` – každá VM dostane unikátne meno `{stack-name}-server-0`, `-1`, atď.

```
Internet (ext-net-154)
    |    |    |   ...
  [VM-0][VM-1][VM-2]  (až 5 VM)
  {stack}-server-%index%

Outputs:
  zoznam_ip --> resource IDs VM (nie IP adresy!)
               ⚠ bug v template: refs() vracia UUIDs,
                 nie IP adresy napriek názvu výstupu
```
