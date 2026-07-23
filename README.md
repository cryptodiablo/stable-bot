# Stable Bot (`sender`)

Standalone-бот для новых токенов в **Stable mainnet** (chain ID `988`). Он покупает подходящие
запуски, выходит по grid / AFK / stop-loss правилам и хранит историю девов в `stable.db`.
Поддерживаются Linux x86_64 и Windows 10/11 x64. Rust, Python и Docker не требуются.

## Быстрый старт — Linux

```bash
mkdir stable-bot && cd stable-bot
curl -fL https://github.com/cryptodiablo/stable-bot/releases/latest/download/stable-bot-linux-x86_64.tar.gz -o stable-bot.tar.gz
tar -xzf stable-bot.tar.gz
chmod +x sender
./sender create-wallet --output ./wallet.key
```

## Быстрый старт — Windows

Откройте PowerShell и выполните:

```powershell
mkdir stable-bot
cd stable-bot
Invoke-WebRequest https://github.com/cryptodiablo/stable-bot/releases/latest/download/stable-bot-windows-x86_64.zip -OutFile stable-bot.zip
Expand-Archive .\stable-bot.zip -DestinationPath . -Force
.\sender.exe create-wallet --output .\wallet.key
```

Команда напечатает адрес кошелька. Пополните его нативным `USDT0` в сети Stable, откройте
`stable.toml` и настройте верхние блоки. Новый конфиг по умолчанию работает безопасно:

```toml
mode = "shadow"
buy_size_mode = "developer"
fixed_buy_amount = 100
developer_buy_cap = 100
minimum_developer_buy = 10
```

Проверьте готовность и запустите:

```bash
./sender check
./sender
```

В Windows те же команды запускаются как `.\sender.exe check` и `.\sender.exe`.

В режиме `shadow` бот только наблюдает. Для реальных сделок установите `mode = "live"`, снова
выполните `./sender check` и запустите `./sender`.

## Конфигурация

`stable.toml` разделён по частоте использования:

- сверху — режим, размер покупки, минимальный buy дева и правила выхода;
- ниже — фильтры дева и метаданных;
- внизу — gas, RPC, таймауты и пути к файлам.

Каждая настройка подписана по-русски прямо в конфиге. Основные режимы суммы покупки:

- `buy_size_mode = "developer"` — повторить initial buy дева, но не выше `developer_buy_cap`;
- `buy_size_mode = "fixed"` — всегда покупать на `fixed_buy_amount`.

`skip_if_position_open = true` запрещает новую покупку, пока предыдущая позиция или кампания не
завершена. Grid задаётся двумя массивами одинаковой длины, а AFK — правилами вида
`"прибыль_в_%:таймаут_в_секундах"`.

Для ускоренного pending lookup и landing вставьте HTTPS endpoint Stable из Alchemy в
`alchemy_rpc_url`. Бот сам использует его первым для lookup/broadcast и выведет из него WSS URL.

## Команды

```bash
./sender                         # обычный запуск
./sender run                     # то же самое явно
./sender check                   # проверка конфига, БД, RPC и контрактов
./sender db-report               # статистика stable.db
./sender update --check          # проверить обновление
./sender update                  # установить новую версию
./sender create-wallet --output ./wallet.key
./sender --help
```

В PowerShell замените `./sender` на `.\sender.exe`; названия команд и настройки одинаковы.

При обычном запуске `sender` сам проверяет GitHub, обновляет только бинарник и перезапускается.
Настройки, ключ, база и позиции при обновлении не меняются.

## Рабочие файлы

- `sender`/`sender.exe` и `stable.toml` входят в соответствующий релиз;
- `wallet.key` создаётся пользователем; в Linux нужны права `0600`, в Windows не выдавайте доступ посторонним пользователям;
- `stable.db`, `name_blacklist.txt` и `stable-state/` создаются автоматически.

Держите эти файлы в одном каталоге и не запускайте две копии на одном кошельке.
