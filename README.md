# Stable Bot (`sender`)

Standalone-бот для автоматического входа в новые токены на **Stable mainnet** (chain ID `988`) и
выхода по grid / AFK / stop-loss правилам. Бот поставляется одним Linux-бинарником, запускается
командой `./sender`, хранит свою историю девов в `stable.db` и не требует установки Rust, Python
или Docker.

## Быстрый старт

Поддерживается Linux x86_64. Скачайте последний релиз:

```bash
mkdir stable-bot && cd stable-bot
curl -fL https://github.com/cryptodiablo/stable-bot/releases/latest/download/stable-bot-linux-x86_64.tar.gz -o stable-bot.tar.gz
tar -xzf stable-bot.tar.gz
chmod +x sender
./sender create-wallet --output ./wallet.key
```

Команда создания кошелька напечатает его публичный адрес. Пополните его нативным `USDT0` в сети
Stable. Приватный ключ хранится в `wallet.key` с правами `0600`.

В поставляемом `stable.ini` включён безопасный режим:

```ini
[BOT]
MODE = SHADOW
PRIVATE_KEY_FILE = ./wallet.key
DATABASE_PATH = ./stable.db
```

Все строки конфигурации подписаны по-русски. Сначала настройте суммы, фильтры и выходы, затем:

```bash
./sender check
./sender
```

В `SHADOW` бот наблюдает, но не отправляет сделки. Для реальной торговли замените
`MODE = SHADOW` на `MODE = LIVE`, снова выполните `./sender check`, после чего запустите
`./sender`.

## Автообновление

Официальный release-бинарник проверяет обновление при каждом обычном запуске. Если опубликована
новая версия, `sender` скачает её, сверит целевую платформу, размер, ELF-заголовок и SHA-256,
атомарно заменит себя и перезапустится. Предыдущая версия останется рядом как `sender.previous`.

Автообновление меняет **только файл `sender`**. Оно не изменяет `stable.ini`, `wallet.key`,
`stable.db`, blacklist и каталог `stable-state`.

```bash
./sender update --check   # только проверить наличие другой сборки
./sender update           # скачать и установить актуальный sender
./sender update --force   # заново установить опубликованную сборку
```

Если GitHub временно недоступен, при обычном старте появится короткое предупреждение, после чего
бот продолжит запуск с уже установленным бинарником.

## Размер покупки

В `stable.ini` доступны два режима:

- `BUY_SIZE_MODE = developer` — повторить первоначальный buy дева, но не выше
  `DEVELOPER_BUY_CAP_STABLE`;
- `BUY_SIZE_MODE = fixed` — всегда покупать на `FIXED_BUY_AMOUNT_STABLE`.

## Фильтры и выходы

Конфиг поддерживает:

- минимальный и максимальный первоначальный buy дева;
- историю запусков и реализованный PnL дева из `stable.db`;
- IPFS image, blacklist имени, повтор картинки/соцсетей/имени и обязательные social-поля;
- фиксированный take-profit, stop-loss и trailing stop;
- многоуровневую grid-продажу;
- AFK-выходы с разными тайм-аутами по достигнутой прибыли;
- отдельные slippage, gas limit и priority fee для buy и sell.

`SKIP_IF_POSITION_OPEN = true` означает, что бот не начнёт новую покупку, пока предыдущая позиция
не закрыта или её покупка ещё не завершена. Это защищает один кошелёк от параллельных кампаний и
nonce-конфликтов.

## Команды

```bash
./sender                         # обычный запуск
./sender run                     # то же самое явно
./sender check                   # проверка конфига, БД, RPC и контрактов
./sender db-report               # накопленная статистика stable.db
./sender create-wallet --output ./wallet.key
./sender --help
```

Консоль показывает только полезные события: готовность RPC, найденный/отфильтрованный launch,
отправку и receipt покупки, комиссию, решение о выходе, продажу и ошибки. Полный durable audit
хранится в `stable-state/events.jsonl`. Для машинного формата используйте `--json-logs`.

## Файлы

- `sender` и `stable.ini` входят в релиз;
- `wallet.key` создаётся пользователем локально и никогда не публикуется;
- `stable.db`, `name_blacklist.txt` и `stable-state/` sender создаёт автоматически при первом
  запуске.

Держите рабочие файлы в одном каталоге и не запускайте две копии на одном кошельке.
