Ограничение частоты запросов
===============================

Чтобы избежать злоупотреблений, вам следует подумать о добавлении ограничения частоты запросов к вашим API. Например, вы можете ограничить
использование каждого метода API, поставив для каждого пользователя ограничение: не более 100 вызовов API в течение 10 минут. Если от пользователя
приходит большее количество запросов в течение этого периода времени, будет возвращаться ответ с кодом состояния 429 (означающий «слишком много запросов»).

Чтобы включить ограничение частоты запросов, *[[yii\web\User::identityClass|класс user identity]]* должен реализовывать интерфейс [[yii\filters\RateLimitInterface]].
Этот интерфейс требует реализации следующих трех методов:

* `getRateLimit()`: возвращает максимальное количество разрешенных запросов и период времени, т.е. `[100, 600]`, что означает
  не более 100 вызовов API в течение 600 секунд.
* `loadAllowance()`: возвращает оставшееся количество разрешенных запросов и соответствующий *UNIX-timestamp*,
  когда ограничение проверялось в последний раз.
* `saveAllowance()`: сохраняет оставшееся количество разрешенных запросов и текущий *UNIX-timestamp*.

Вы можете также использовать два столбца в таблице пользователей для записи количества разрешенных запросов и *отметки времени*.
`loadAllowance()` и `saveAllowance()` могут быть реализованы как чтение и сохранение значений, относящихся к текущему аутентифицированному пользователю,
в этих двух столбцах. Для улучшения производительности вы можете подумать о том, чтобы хранить эту информацию
в кэше или каком-либо NoSQL-хранилище.

Так как *identity class* реализует требуемый интерфейс, Yii будет автоматически использовать [[yii\filters\RateLimiter]],
настроенный как фильтр действий в [[yii\rest\Controller]], для выполнения проверки на количество разрешенных запросов. Если ограничение на количество
запросов будет превышено, выбрасывается исключение [[yii\web\TooManyRequestsHttpException]]. Вы можете настроить ограничитель частоты запросов
в ваших классах REST-контроллеров следующим образом:

```php
public function behaviors()
{
    $behaviors = parent::behaviors();
    $behaviors['rateLimiter']['enableRateLimitHeaders'] = false;
    return $behaviors;
}
```

Когда ограничение частоты запросов включено, по умолчанию каждый ответ будет возвращаться со следующими HTTP-заголовками, содержащими
информацию о текущих ограничениях количества запросов:

* `X-Rate-Limit-Limit`: максимальное количество запросов, разрешенное в течение периода времени;
* `X-Rate-Limit-Remaining`: оставшееся количество разрешенных запросов в текущем периоде времени;
* `X-Rate-Limit-Reset`: количество секунд, которое нужно подождать до получения максимального количества разрешенных запросов.

Вы можете отключить эти заголовки, установив свойство [[yii\filters\RateLimiter::enableRateLimitHeaders]] в значение false,
как показано в примере кода выше.