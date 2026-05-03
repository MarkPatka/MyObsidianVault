`nano start-mtproxy.sh`

## Short version
```bash
#!/bin/bash
CONTAINER_NAME="mtproto-proxy"
PORT="443"
FAKE_DOMAIN="ya.ru"
if ss -tuln | grep -q ":${PORT} "; then
    for alt_port in 8443 8444 8445; do
        if ! ss -tuln | grep -q ":${alt_port} "; then
            PORT=$alt_port
            break
        fi
    done
fi
DOMAIN_HEX=$(echo -n $FAKE_DOMAIN | xxd -ps | tr -d '\n')
RANDOM_HEX=$(openssl rand -hex 15 | cut -c1-$((30-${#DOMAIN_HEX})))
SECRET="ee${DOMAIN_HEX}${RANDOM_HEX}"
sudo docker stop ${CONTAINER_NAME} 2>/dev/null; sudo docker rm ${CONTAINER_NAME} 2>/dev/null
sudo docker run -d --name ${CONTAINER_NAME} --restart unless-stopped -p ${PORT}:443 -e SECRET="${SECRET}" telegrammessenger/proxy
sleep 3
SERVER_IP=$(curl -s ifconfig.me)
echo "tg://proxy?server=${SERVER_IP}&port=${PORT}&secret=${SECRET}"
```

## Full script
source: [Настраиваем MTProto прокси с Fake TLS за 5 минут](https://habr.com/ru/articles/1010942/)
```bash
#!/bin/bash

# Цвета для красивого вывода
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

CONTAINER_NAME="mtproto-proxy"
PORT="443"
FAKE_DOMAIN="ya.ru"  # Фиксированный домен для Fake TLS

echo "🚀 Запуск MTProto прокси с Fake TLS"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo -e "📌 Используем домен: ${BLUE}${FAKE_DOMAIN}${NC}"

# Генерируем секрет для Fake TLS
echo -n "🔑 Генерация Fake TLS секрета... "

# Получаем hex домена ya.ru
DOMAIN_HEX=$(echo -n $FAKE_DOMAIN | xxd -ps | tr -d '\n')
echo -e "\n   Hex домена: ${DOMAIN_HEX}"

# Дополняем случайными символами до 30 символов
DOMAIN_LEN=${#DOMAIN_HEX}
NEEDED=$((30 - DOMAIN_LEN))
RANDOM_HEX=$(openssl rand -hex 15 | cut -c1-$NEEDED)

# Собираем секрет
SECRET="ee${DOMAIN_HEX}${RANDOM_HEX}"

echo -e "   Случайное дополнение: ${RANDOM_HEX}"
echo -e "   Секрет: ${YELLOW}${SECRET}${NC}"
echo "   Длина: ${#SECRET} символов"

# Проверяем, свободен ли порт 443
echo -n "🔍 Проверка порта ${PORT}... "
if ss -tuln | grep -q ":${PORT} "; then
    echo -e "${YELLOW}порт занят${NC}"
    # Ищем альтернативный порт
    for alt_port in 8443 8444 8445; do
        if ! ss -tuln | grep -q ":${alt_port} "; then
            PORT=$alt_port
            echo "   Используем порт: ${PORT}"
            break
        fi
    done
else
    echo -e "${GREEN}свободен${NC}"
fi

# Останавливаем старый контейнер, если есть
echo -n "🛑 Остановка старого контейнера... "
sudo docker stop ${CONTAINER_NAME} >/dev/null 2>&1
sudo docker rm ${CONTAINER_NAME} >/dev/null 2>&1
echo -e "${GREEN}готово${NC}"

# Запускаем официальный прокси от Telegram
echo -n "📦 Запуск контейнера... "
sudo docker run -d \
  --name ${CONTAINER_NAME} \
  --restart unless-stopped \
  -p ${PORT}:443 \
  -e SECRET="${SECRET}" \
  telegrammessenger/proxy > /dev/null 2>&1

# Проверяем результат
sleep 3
if sudo docker ps | grep -q ${CONTAINER_NAME}; then
    SERVER_IP=$(curl -s ifconfig.me)
    
    echo -e "${GREEN}✅ УСПЕШНО${NC}"
    echo ""
    echo "📊 ИНФОРМАЦИЯ ДЛЯ ПОДКЛЮЧЕНИЯ:"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "🌐 Сервер: ${SERVER_IP}"
    echo "🔌 Порт: ${PORT}"
    echo "🔑 Секрет: ${SECRET}"
    echo "🌐 Fake TLS домен: ${FAKE_DOMAIN}"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "🔗 Ссылка для Telegram (нажмите для автоподключения):"
    echo -e "${GREEN}tg://proxy?server=${SERVER_IP}&port=${PORT}&secret=${SECRET}${NC}"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    
    # Сохраняем конфигурацию
    cat > ~/mtproto_config.txt << EOF
SERVER=${SERVER_IP}
PORT=${PORT}
SECRET=${SECRET}
DOMAIN=${FAKE_DOMAIN}
LINK=tg://proxy?server=${SERVER_IP}&port=${PORT}&secret=${SECRET}
EOF
    echo "✅ Конфигурация сохранена в ~/mtproto_config.txt"
    
    # Показываем последние логи
    echo ""
    echo "📋 Логи контейнера:"
    sudo docker logs --tail 5 ${CONTAINER_NAME}
else
    echo -e "${RED}❌ ОШИБКА${NC}"
    sudo docker logs ${CONTAINER_NAME}
fi
```

## Make executable and launch
```bash
chmod +x start-mtproxy.sh
./start-mtproxy.sh
```

### Как это работает

Скрипт делает следующее:

1. Предлагает выбрать домен для маскировки трафика
2. Генерирует секретный ключ с префиксом `ee` (признак Fake TLS)
3. Проверяет, свободен ли порт 443 (стандартный HTTPS порт)
4. Запускает официальный Docker-образ прокси от Telegram
5. Выдаёт готовую ссылку для подключения
### Подключение в Telegram

#### На телефоне:

1. Нажмите на сгенерированную ссылку `tg://...`
2. Telegram сам предложит активировать прокси
3. Нажмите "Добавить прокси" и готово!
#### Вручную:

- **На мобильных устройствах:** Настройки → Данные и память → Настройки прокси → Добавить прокси → MTProto
- **На десктопе:** Настройки → Продвинутые настройки → Тип соединения → Использовать собственный прокси → Добавить прокси → MTProto

Введите IP сервера, порт (обычно 443) и секретный ключ, который сгенерировал скрипт.