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

