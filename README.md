# Yii2 Sberbank payment module

### Установка через композер
```
composer require pantera-digital/yii2-sberbank-pay "@dev"
```

### Запустить миграции
```
php yii migrate --migrationPath=@pantera/yii2/pay/sberbank/migrations
```

### Настройка, добавить в config/main.php

```
'modules' => [
    'sberbank' => [
        'class' => 'pantera\yii2\pay\sberbank\Module',
        'components' => [
            'sberbank' => [
                'class' => pantera\yii2\pay\sberbank\components\Sberbank::className(),
                // время жизни инвойса в секундах (по умолчанию 20 минут - см. документацию Сбербанка)
                // в это примере мы ставим время 1 неделю, т.е. в течение этого времени покупатель может произвести оплату
                'sessionTimeoutSecs' => 60 * 60 * 24 * 7,
                // логин api мерчанта
                'login' => 'ваш логин',
                // пароль api мерчанта
                'password' => 'ваш пароль',
                // использовать тестовый режим (по умолчанию - нет)
                'testServer' => false,
            ],
        ],
        // страница вашего сайта с информацией об успешной оплате
        'successUrl' => '/paySuccess',
        // страница вашего сайта с информацией об НЕуспешной оплате
        'failUrl' => '/payFail',
        // обработчик, вызываемый по факту успешной оплаты
        'successCallback' => function($invoice){
            // какая-то ваша логика, например
            $order = \your\models\Order::findOne($invoice->id);
            $client = $order->getClient();
            $client->sendEmail('Зачислена оплата по вашему заказу №' . $order->id);
            // .. и т.д.
        }
    ],
]
```

### Создание заказа

В вашем контроллере после сохранения заказа, либо на событие создания заказа вам необходимо создать инвойс, передав в него номер и сумму вашего заказа:

```
// ...здесь какая-то ваша логика по сохранению заказа, например это объект $order

// создаем и сохраняем инвойс, передаем в него номер и сумму вашего заказа
$invoice = \pantera\yii2\pay\sberbank\models\Invoice::addSberbank($order->id, $order->price);
```

Далее для перенаправления пользователя на шлюз оплаты Сбербанка вам нужно выдать пользователю ссылку (либо автоматически перенаправить его) на url:

```
\yii\helpers\Html::a('Оплатить заказ', ['/sberbank/default/create', 'id' => $invoice->id /* id инвойса */])
```

При этом при переходе пользователя по этой ссылке (либо автоматическом перенаправлении) будет произведено обращение к API сбербанка для создания инвойса у них в системе, и перенаправление уже на платежную форму Сбербанка.

### Статусы инвойсов
```
I - initial, инвойс создан
S - success, успешно оплачен
```
