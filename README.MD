# TrinityCore WotLK Classic 3.4.3.54261 (WDC4) — FULL HOW‑TO

In this guide, we included everything required to setup a wotlk classic 3.4.3 WoW server using TrinityCore, including:
- what we cloned / downloaded (repos + DB),
- how we built the project,
- how we packaged a deploy folder,
- how we fixed the real errors we hit (and why),
- and the final login steps (TLS cert + REST/SRP).

NOTE: There's no illegal "stuff" here. everything included in this guide, is completely open-source and community made.

## Requirements
- https://github.com/TrinityCoreLegacy/TrinityCore/tree/3.4.3 - Main core repo
- https://github.com/TrinityCore/TrinityCore/releases - TDB files required for worldserver
- https://github.com/haphert/TrinityCore_wotlk_classic_continued - some helpful patches
- A MySQL sever (version 8+)
- C/CPP Toolchain with following libraries/Tools: CMake - Git - Make - MYSQL (8+) - Boost (1.83.0) - OpenSSL (3.x)

## Common Errors / Problems
- `invalid literal suffix 'h'` or `std::chrono::high_resolution_clock` not found
### Fix
- `TrinityCore/src/common/Utilities/Duration.h`: add `<chrono>`
- `TrinityCore/src/server/database/Updater/UpdateFetcher.cpp`: add `<chrono>`

## 0 - Build the core using cmake
- First, create a build folder:
```bash
mkdir -p build/
```
- After doing so, execute the following command:
```bash
cd build/
cmake ../
```
- This command will then provide required files to build the project. after the execution, you need to build the project using make:
```bash
make
make install
```
- You can optionally also provide a -j argument to use your full CPU power to build more efficiently.

## 1 - Setup the database
- Execute the following queries on your MYSQL server
```mysql
mysql -u root -p -e "CREATE DATABASE auth DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -p -e "CREATE DATABASE characters DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -p -e "CREATE DATABASE world DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -p -e "CREATE DATABASE hotfixes DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -p -e "CREATE USER 'trinity'@'localhost' IDENTIFIED BY 'trinity'; GRANT ALL PRIVILEGES ON *.* TO 'trinity'@'localhost'; FLUSH PRIVILEGES;"
```
- Now with that queries being done, you need TDB_FULL file. this file could be downloaded from Trinity Core repository. 
- https://github.com/TrinityCore/TrinityCore/releases/download/TDB343.24081/TDB_full_343.24081_2024_08_17.7z
```mysql
mysql -u root -p world < "TDB_full_world_343.24081_2024_08_17.sql"
```
- After executing the queries, you also need to import 3 .mysql files. this files are provided / executed by trinity core itself, but just in case:
```mysql
mysql -u root -p auth < "SQL\base\auth_database.sql"
mysql -u root -p characters < "SQL\base\characters_database.sql"
mysql -u root -p hotfixes < "SQL\TDB\TDB_full_hotfixes_343.24081_2024_08_17.sql"
```
## 2 - Setup the config files
- Depending on your environment, you would have your worldserver and bnetserver binaries and configs in /usr/local/bin / /usr/local/etc, but generally, you need to edit configurations in /usr/local/etc or SERVER_DIR/etc/worldserver.conf or SERVER_DIR/worldserver.conf
### in your worldserver.conf, edit the following variables:
- `DataDir` must point to `..\Data` (or absolute path)
- **keep** `Updates.EnableDatabases = 0`
### in your bnetserver.conf, edit the following variables:
- `Updates.EnableDatabases = 0`

## 3 - install world files
- you should use provided tools in build/ folder to get data files, including maps, camera, vmaps, etc. and provide them in your Data folder.

## 4 - Ensure firewall doesn't block server ports
- Execute the following commands to configure your firewall:
```bash
sudo ufw allow 1119/tcp
sudo ufw allow 8081/tcp
sudo ufw allow 8085/tcp
```
## 5 - TLS configuration
### 4.5) TLS certificate for bnetserver (self‑signed with SAN IP)
- The client/launcher is sensitive to the certificate presented on 1119. We fixed our BLZ issues by:
- generating a cert whose **CN is the public IP**, and
- including the IP in **SAN** (Subject Alternative Name),
- and forcing absolute paths in config (otherwise TC may serve the default cert).
- Execute the following commands in your data folder:
```bash
openssl req -x509 -newkey rsa:2048 -keyout bnetserver.key.pem -out bnetserver.cert.pem -days 365 -nodes \
  -subj "/CN=YOUR-IP" \
  -addext "subjectAltName=IP:YOUR-IP,DNS:localhost,IP:127.0.0.1"
```
- Replace YOUR-IP with your server IP.
- In `bnetserver.conf` set absolute paths:
```toml
CertificatesFile = "/opt/trinitycore/bin/bnetserver.cert.pem"
PrivateKeyFile   = "/opt/trinitycore/bin/bnetserver.key.pem"
```
### Note on /bnetserver/login/ returning: `"srp_url":"https://127.0.0.1:8081/bnetserver/login/srp/"`
- Fix:
- in `bnetserver.conf`:
```toml
LoginREST.ExternalAddress=YOUR-IP
LoginREST.LocalAddress=YOUR-IP
```

