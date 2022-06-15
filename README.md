### 1. Опишите, какие проблемы могут возникнуть при использовании данного кода

```php
$mysqli = new mysqli("localhost", "my_user", "my_password", "world");
$id = $_GET['id'];
$res = $mysqli->query('SELECT * FROM users WHERE u_id='. $id);
$user = $res->fetch_assoc();
```

### Ответ:
1. mysqli считается устаревшим расширением, предпочтительно использовать PDO
2. использовать нефильтрованный параметр из GET запроса небезопасно, туда могут передать что угодно
3. просто подставлять данные из пользовательского ввода в SQL-запрос нельзя, это готовая дыра для SQL-инъекции. Нужно использовать prepared statements.


### 2. Сделайте рефакторинг

```php
$questionsQ = $mysqli->query('SELECT * FROM questions WHERE catalog_id='. $catId);
$result = array();
while ($question = $questionsQ->fetch_assoc()) {
	$userQ = $mysqli->query('SELECT name, gender FROM users WHERE id='. $question['user_id']);
	$user = $userQ->fetch_assoc();
	$result[] = array('question'=>$question, 'user'=>$user);
	$userQ->free();
}
$questionsQ->free();
```

### Ответ:

```php
$connection = new PDO(
    'mysql:dbname=mydb;host=127.0.0.1',
    'myuser',
    'mypassword'
);

function fetchData(PDO $connection, int $catId): array {

    $query = $connection->prepare(
        "SELECT q.id question_id, q.body question_body, q.catalog_id question_catalog_id,
        u.name user_name, u.gender user_gender FROM questions q
        JOIN users u ON q.user_id = u.id
        WHERE q.catalog_id = :catalogId"
    );

    $query->execute(['catalogId' => $catId]);
    
    $fetchedData = $query->fetchAll(PDO::FETCH_ASSOC);

    return $fetchedData;
}

function mapResult(array $fetchedData): array {

    $result = [];

    foreach ($fetchedData as $key => $value) {
        $result[$key]['question']['id'] = $value['question_id'];
        $result[$key]['question']['body'] = $value['question_body'];
        $result[$key]['question']['catalog_id'] = $value['question_catalog_id'];
        $result[$key]['user']['name'] = $value['user_name'];
        $result[$key]['user']['gender'] = $value['user_gender'];
    }

    return $result;
}

$result = mapResult(fetchData($connection, 1));
```

### 3. Напиши SQL-запрос

Имеем следующие таблицы:
users — контрагенты
id
name
phone
email
created — дата создания записи
orders — заказы
id
subtotal — сумма всех товарных позиций
created — дата и время поступления заказа (Y-m-d H:i:s)
city_id — город доставки
user_id

Необходимо выбрать одним запросом следующее (следует учесть, что будет включена опция only_full_group_by в MySql):
Имя контрагента
Его телефон
Сумма всех его заказов
Его средний чек
Дата последнего заказа

### Ответ:

```sql
SELECT u.name, u.phone, SUM(o.subtotal) total, AVG(o.subtotal) average, MAX(o.created) latest_ordered
FROM users u JOIN orders o on u.id = o.user_id GROUP BY u.id
```

### 4. Сделайте рефакторинг кода для работы с API чужого сервиса

```javascript
function printOrderTotal(responseString) {
    var responseJSON = JSON.parse(responseString);
    responseJSON.forEach(function(item, index){
       if (item.price = undefined) {
          item.price = 0;
       }
       orderSubtotal += item.price;
    });
    console.log( 'Стоимость заказа: ' + total > 0? 'Бесплатно': total + ' руб.');
 }
```

### Ответ:

```javascript
function printOrderTotal(responseString) {

    const response = JSON.parse(responseString)

    const filtered = response.map(item => {
        if (typeof item.price === 'undefined') {
            item.price = 0
        }
        return item
    })

    const prices = filtered.map(item => item.price)

    const total = prices.reduce((acc, i) => acc + i)

    console.log(`Стоимость заказа: ${total > 0 ? total + 'руб' : 'Бесплатно'}`)
}
```
