Ревью на проект Hangman для [@Olegarhische](https://t.me/zhukovsd_it_chat/53243/293203)

**Что выглядит неплохо**
=
 - Понятная структура проекта
 - Хороший, в большинстве случаев, нейминг
 - Использование immutable классов таких как record

**Недостатки реализации**
=
 ! Нет Readme, непонятно как запускать проект
1. Класс [ServletContextListener](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/contextListener/ServletContextListener.java#L11) уже существует с таким названием, и даже более того — ты прям здесь его уже импортируешь. Так делать не надо по нескольким причинам:
 - Ты не создаёшь свой какой-то класс, который будет служить той же цели, только со специфичиской логикой — ты его переиспользуешь готовый, а значит лучше было бы его назвать CurrencyExchangeServletContextListener
 - Представим, что тебе нужно будет использовать ещё несколько подобных ContextListener-ов из других фреймворков/библиотек, тебе будет гораздо легче запутаться в импортах и потом будешь сломя голову искать, а в чём же ошибка
2. Не нужно [здесь](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dao/Dao.java#L8-L7) лишний пробел :)
3. Хорошо, что существует интерфейс [Dao](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dao/Dao.java#L5-L9), но тут же дальше у тебя нарушение принципа DRY. О чём я говорю:
  [Здесь](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dao/JdbcCurrencyDao.java#L58-L87) и [здесь](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dao/JdbcExchangeRateDao.java#L81-L110) ты имплементируешь абсолютно одинаковую логику и пишешь один и тот же код. Это не критично сейчас, но подумай как это можно улучшить с помощью Generics.
4. Класс [JdbcCurrencyDao](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dao/JdbcCurrencyDao.java#L17)
 в целом выглядит неплохо, но есть одна очень большая проблема. 
 - Утечка памяти во всех случаях работы с базой.
 [findByCode](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dao/JdbcCurrencyDao.java#L43-L56),
[save](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dao/JdbcCurrencyDao.java#L58-L72),
[findAll](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dao/JdbcCurrencyDao.java#L74-L87) — все они содержат одну и ту же ошибку, которая в целом критичная. 
Ты открывая [PreparedStatement](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dao/JdbcCurrencyDao.java#L78) пулл его не закрываешь в следствие чего будет убегать память. В каждом из этих случаев нужно либо использовать try-with-resource, как ты уже сделал выше, либо закрыть connection:
````
    @Override
    public List<Currency> findAll() {
        List<Currency> currencies = new ArrayList<>();
        try (Connection connection = DataSource.getConnection()) {
            PreparedStatement statement = connection.prepareStatement(FIND_ALL);
            ResultSet resultSet = statement.executeQuery();
            while (resultSet.next()) {
                currencies.add(buildCurrency(resultSet));
            statement.close; <--- Говорю об этом
            return currencies;
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
````
 - Почему было принято решение использовать Singleton? Как по мне здесь он не нужен прямо совсем
 - Ну и мне непонятно, почему ты сделал кастомные исключение, но для `findAll()` решил проигнорировать и просто бросил Runtime :)

5. Класс [JdbcExchangeRateDao](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dao/JdbcExchangeRateDao.java#L16) содержит те же критические ошибки, что и класс `JdbcCurrencyDao`, только здесь ты не создал кастомное исключение ещё и для `findRate()` :)
6. Зачем тебе классы [CurrencyRequestDto](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dto/CurrencyRequestDto.java#L3) и [ExchangeRateRequestDto](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/dto/ExchangeRateRequestDto.java#L7)? Удали их, они неиспользуемые, а значит им не нужно быть в проекте
7. Это написал не ты, а AI. Ты всё же копировал и использовал AI-шку. У меня были подозрения ещё вот на [этом](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/contextListener/ServletContextListener.java#L11) моменте, а вот [тут](https://github.com/Olegarh86/Exchanger_ferever/blob/090e3db2ae97ea4633878c97d48ea50cdbab9692/src/main/java/mappers/ExchangerMapper.java#L11) уже сомнений не осталось. Дальше смотреть не хочу, потому что я хотел, чтобы написал это ты сам.
<img width="1293" height="386" alt="image" src="https://github.com/user-attachments/assets/463dbcec-db82-44de-a549-57f9233e1b34" />
<img width="1157" height="381" alt="image" src="https://github.com/user-attachments/assets/5370f63c-071e-418c-8560-6e0fa0a7a0a6" />

8. Overall Ai-шка написала неплохо, но хотелось бы увидеть твою работу.
