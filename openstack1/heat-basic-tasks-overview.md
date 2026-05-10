# OpenStack Heat – Prehľad úloh

---

## Úloha 1 — `task1.yml`
**Dve VM v spoločnej externej sieti**

Skript vytvorí security group (ICMP + SSH), dva Neutron porty v sieti `ext-net-154` a dve VM (`vm1`, `vm2`) napojené každá na vlastný port. Žiadna interná sieť – obe VM sú priamo v externej sieti.

```
┌─────────────────────────────────────┐
│          ext-net-154                │
│                                     │
│   ┌──────────┐    ┌──────────┐      │
│   │   vm1    │    │   vm2    │      │
│   │ (cirros) │    │ (cirros) │      │
│   └──────────┘    └──────────┘      │
│      port1           port2          │
└─────────────────────────────────────┘

Security group: ICMP + TCP/22
```

---

## Úloha 2 — `task2.yml`
**VM router + interná VM za ním**

Skript vytvorí internú sieť `net-is1` (subnet `192.168.50.0/24`), security group (ICMP + SSH), a dve VM. `vm-router` má dve sieťové rozhrania – jedno v externej a jedno v internej sieti, pričom pri štarte povolí IP forwarding. `vm-internal` je pripojená iba do internej siete.

```
                  ext-net-154
                       │
              ┌────────┴────────┐
              │    vm-router    │
              │    (cirros)     │
              │  IP forwarding  │
              └────────┬────────┘
                       │
          ┌────────────────────────┐
          │  net-is1               │
          │  192.168.50.0/24       │
          │                        │
          │  ┌─────────────────┐   │
          │  │   vm-internal   │   │
          │  │    (cirros)     │   │
          │  └─────────────────┘   │
          └────────────────────────┘

Security group: ICMP + TCP/22
```

---

## Úloha 3 — `task3.yml`
**Router VM + web server s obmedzeným security group**

Skript vytvorí internú sieť `net-is2` (subnet `10.0.1.0/24`), security group s obmedzeným SSH (iba z UNIZA siete `158.193.0.0/16`) a povoleným HTTP (port 80). `router-vm` má dve rozhrania, povolí smerovanie a nastaví NAT (masquerade). `web-vm` je v internej sieti a automaticky nainštaluje Apache2.

```
  Internet
     │
  ext-net-154
     │
┌────┴──────────────┐
│    router-vm      │
│  (debian-12-kis)  │
│  ip_forward + NAT │
└────┬──────────────┘
     │
┌────────────────────────────┐
│  net-is2  (10.0.1.0/24)    │
│                            │
│  ┌──────────────────────┐  │
│  │       web-vm         │  │
│  │   (debian-12-kis)    │  │
│  │   Apache2 (HTTP:80)  │  │
│  └──────────────────────┘  │
└────────────────────────────┘

Security group:
  SSH  → len 158.193.0.0/16 (UNIZA)
  HTTP → 0.0.0.0/0
  ICMP → 0.0.0.0/0

⚠ web-vm vyžaduje manuálne nastavenie default route cez router-vm
```
