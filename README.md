# RZHD
Проект посвящен прогнозированию зарождающегося грузового вагонопотока для разработки системы планирования на каждые 4 часа с горизонтом планирования 48 часов.

**Описание функций:**

_Строка 8._ Функция _ **filter\_needed\_wagon\_operations** _ - получает на вход датафрейм со всеми операциями, происходившими с вагонами, список кодов операций включения в поезд и список кодов операций загрузки вагона. На выходе выдает очищенный от лишних операций датафрейм, готовый к дальнейшей работе.

Необходимо оставить только последние операции включения вагона в поезд, которым предшествует погрузка/выгрузка, при условии, что после включения была операция отправления поезда. Данная методология очистки утверждена и согласована заказчиком.

Подробно опишем реализованный алгоритм. Первым шагом убираем из датасета все неинтересующие нас операции, оставляем только коды операций отправки поезда, погрузки/выгрузки, включения вагона в поезд. Затем сортируем датафрейм по направлению, номеру вагона и дате в порядке убывания и начинаем итерировать по нему.

Так как мы проходим от самых новых событий к самым старым, то начинаем искать с операции отправления поезда. Если находим, то обновляем индикаторы: departure\_indicator= 1, included\_indicator= 0, а также следим за тем, что операции мы смотрим по тому же вагону (current\_wagon) что и отправление поезда.

Далее если находим операцию включения в поезд, при этом ранее была операция отправления поезда, вагон тот же самый и до этого включения в поезд не было, то сохраняем эту операцию на потенциальное добавление в финальный датафрейм и обновляем индикатор включения в поезд included\_indicator= 1.

Если после этого мы для того же вагона находим операцию погрузки/выгрузки и операция включения была, то эту операцию включения с прошлого шага, сохраненную в переменной chosen\_operation, мы добавляем в финальный датафрейм. Затем обнуляем все индикаторы, чтобы продолжить поиск операций для других вагонов.

_Строка 24._ Функция_ **optimize\_ARIMA** _ - функция настройки оптимальных параметров (p, d, q) для модели.

_Строка 43._ Функция _ **is\_holiday** _ - присваивает 1, если дата выпадает на выходной день (согласно календарю РФ), в ином случае присваивает 0

Функция _ **add\_holiday\_col** _ - добавляет новую фичу в наш датасет, праздничный (1) / рабочий (0) день

_Строка 47_ и _строка 55_. Функция _ **xgb\_mape** _ - используется для вычисления средней абсолютной квадратичной ошибки

_Строка 77_. Функция _ **fill\_0** _ - заполняет пропуски нулями в данных о количестве вагонов, отправленных со станции в промежутки времени, когда отправлений не было

_Строка 78_. Функция _ **make\_features** _ - добавление фичей (час, день, день недели)

Функция _ **xgb\_mape** _ - используется для вычисления средней абсолютной квадратичной ошибки

Функция _ **split** _ - создание train/test/val

Функция _ **make\_prediction** _ - на вход получает train/test/val, параметры модели xgb.XGBRegressor(max\_depth= 40, learning\_rate=0.01, n\_estimators=4000, silent=True, objective=&#39;reg:linear&#39;, booster=&#39;gbtree&#39;, subsample= 0.6, colsample\_bytree= 0.9, colsample\_bylevel= 1, reg\_lambda= 20 , random\_state=42 , seed= 1, importance\_type=&#39;gain&#39;), возвращает предсказание
