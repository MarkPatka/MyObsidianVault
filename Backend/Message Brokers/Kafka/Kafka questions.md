1) Почему replication factor должен быть как минимум на единицу меньшим чем количество реплик? Почему нельзя сделать все реплики ISR? Как из-за этого может не выполниться запись данных в реплику?
2) Как происходит отправка сообщения в Kafka?
	- Producer (send message) -> Происходит установка [[3. Kafka Key Concepts|гарантии доставки]] (acks 0 (none), 1 (leader) или -1 (all)) и [[3. Kafka Key Concepts|семантики доставки]] (at most once, at least once, exactly once (idempotence))
	- Fetch metadata -> синхронная, блокирующая, дорогая операция (может занимать до 60 сек. по умолчанию), обращавшаяся к zookeeper (в ранних версиях), собирающая информацию о кластере и топике. ==Как это происходит в новых версиях с переходом на KRaft?==
	- Serialize message -> у producer указывается key.serializer и value.serializer (например string serializer, если сообщение достаточно сериализовать просто в строку)
	- Define partition -> explicit partition (явное указание партиции), автоматическое определение (round-robin: 0-1-2-3...), key-defined (key_hash % n) по ключу. 
	- Compress message
	- Accumulate batch (via batch.size and linger.ms) ==Если происходит накопление batch сообщениями, то что произойдет если брокер отвалится до записи этого batch?== отправка может быть осуществлена либо по превышении batch size, либо по достижении времени linger.ms, а также, если для одного брокера созданы 2 партиции и для каждой создан batch, и суммарный объем этих batch >= batch.size, происходит запись
	- Запись bathes в Leader-партиции. 
3)   
  

