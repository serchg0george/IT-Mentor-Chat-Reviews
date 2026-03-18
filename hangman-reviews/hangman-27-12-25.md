Ссылка на проект - https://github.com/nadillustrator/HangmanGame


**Хорошо**:
-
  - Класс Main содержит только точку входа в программу
  - Разделение на модели и сервисы
  - Нейминг переменных и методов

**Что можно улучшить**:
-

1. Класс GameService содержит не только логику, которая относится к игре, но и логику которая ему не принадлежит, чем нарушает Single Responsibility Principle
   Например было бы уместнее вместо того, чтобы валидировать Input напрямую здесь, вынести валидацию в отдельный класс GameValidator

```java
public class GameValidator {

    private boolean isInputValueValid(String inputValue, Game game) {
        if (isOneSymbol(inputValue)) {
            System.err.println("Нужно ввести один символ");
            return false;
        } else if (isLetterWasNamedBefore(inputValue, game)) {
            System.err.println("Вы уже называли " + inputValue);
            return false;
        } else if (!isLetterCyrillicSymbol(inputValue)) {
            System.err.println("Введите букву из кириллицыыы");
            return false;
        }
        return true;
    }

    private static boolean isOneSymbol(String inputValue) {
        return inputValue.length() > 1;
    }

    private static boolean isLetterWasNamedBefore(String inputValue, Game game) {
        return game.getNamedLetters().contains(inputValue);
    }

    public boolean isLetterCyrillicSymbol(String inputValue) {
        return Character.UnicodeBlock.of(inputValue.charAt(0)).equals(Character.UnicodeBlock.CYRILLIC);
    }
  }
```
****
2. Если [здесь](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L89) нет необходимости использовать Boolean 
вместо boolean, то предпочтительнее использовать примитивы. Можешь почитать, что такое wrapper классы и какая разница между такими классами и примитивами
```java
    private Boolean isInputValueValid(String inputValue, Game game) {
        if (isOneSymbol(inputValue)) {
            System.err.println("Нужно ввести один символ");
            return false;
        } else if (isLetterWasNamedBefore(inputValue, game)) {
            System.err.println("Вы уже называли " + inputValue);
            return false;
        } else if (!isLetterCyrillicSymbol(inputValue)) {
            System.err.println("Введите букву из кириллицыыы");
            return false;
        }
        return true;
    }
```
****
3. Так же и [здесь](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L129-L132), лучше использовать примитив для того, чтобы избежать ненужный боксинг/анбоксинг
```java
        if (isOneSymbol(inputValue)) {
            System.err.println("Нужно ввести один символ");
            return false;
        } else if (isLetterWasNamedBefore(inputValue, game)) {
            System.err.println("Вы уже называли " + inputValue);
            return false;
        } else if (!isLetterCyrillicSymbol(inputValue)) {
            System.err.println("Введите букву из кириллицыыы");
            return false;
        }
```
****
4. Метод [fillInWordList()](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L26-L39) не принадлежит классу GameService, метод пополняет запас слов, что по-сути является вспомогательным методом который можно вынести в отдельный класс GameHelper.
Так же, как в этот же хелпер класс можно отправить методы [changePicture() и checkVictory()](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L69-L87), а так же [showNewGameMenu()](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L134-L138)
Как и метод [formDisplayWord()](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L156-L158) который по-сути не является сервисным, это вспомогатильный метод
****
5. Я бы поменял метод [checkVictory](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L75-L87) следующим образом:
а) Изменить имя на isVictory()
б) Вместо типа void добавил бы возвращаемый тип boolean, чтобы после его передавать непосредственно в game,
потому что сейчас если ты расширишь логику и твой проект обрастёт классами, то тебе придётся очень долго понимать, а откуда у тебя вообще приходят значения о том победа это или нет?
```java
    private static boolean checkVictory(Game game) {
        if (game.getHiddenWord().equals(game.getDisplayWord())) {
            System.out.println("Поздравляю! Вы выиграли! Загаданное слово: " + game.getDisplayWord());
            game.setVictory(true);
        }
        if (game.getPicture().equals(Picture.getPictures().get(6))) {
            System.out.println();
            System.out.println(game.getPicture());
            System.out.println("Вы проиграли. Загаданное слово: " + game.getHiddenWord());
            System.out.println();
            game.setVictory(false);
        }
    }
```
****
6. В классе MenuService метод [startGame()](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/MenuService.java#L15-L30) принадлежит скорее к GameService, поскольку он содержит логику запуска самой игры, и он совершенно точно не относится к меню
****
7. У меня есть большие подозрения, что проект писался не без помощи ИИ. И на данном этапе это скорее минус, чем плюс. Навело меня на мысль [вот это](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L29-L32):
```java
            words = lines
                    .map(String::trim)
                    .filter(s -> s.length() > 5)
                    .collect(Collectors.toList());
```
****
8. В методе [startGame()](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/MenuService.java#L15-L30) можно спокойно убрать цикл while и логика программы не изменится, зачем он там нужен вообще?
```java
    public static void startGame() {
        showStartMenu();
        String choice = scanner.next();
        GameService gameService = new GameService();
        if (choice.equals(PLAY)) {
            gameService.playNewGame();
        } else {
            System.out.println("ИГРА ЗАВЕРШЕНА");
        }

        scanner.close();
    }
```
9. В этом же методе startGame() лучше не создавать GameService инстанс, а вынести эту зависимость в конструктор вот так:
```java
    // Инициализация GameService-а ушла сюда. Можешь пока не задумываться почему, но если интересно, прочитай про Dependency Injection
    public final GameService gameService;

    public MenuService(GameService gameService) {
        this.gameService = gameService;
    }

    public void startGame() {
        showStartMenu();
        String choice = scanner.next();

        if (choice.equals(PLAY)) {
            gameService.playNewGame();
        } else {
            System.out.println("ИГРА ЗАВЕРШЕНА");
        }

        scanner.close();
    }
```
А ещё в этом же классе соверешнно неясно зачем нужны static модификаторы, можно спокойно обойтись и без них и тогда наш MenuService и Main будут выглядеть вот так:
```java
public class MenuService {
    public static final String PLAY = "1";
    public static final Scanner scanner = new Scanner(System.in);

    public final GameService gameService;

    public MenuService(GameService gameService) {
        this.gameService = gameService;
    }

    public static  void showStartMenu() {
        System.out.println("Начать новую игру: введите " + PLAY);
        System.out.println("Выход:             введите любое другое значение");
    }

    public void startGame() {
        showStartMenu();
        String choice = scanner.next();

        if (choice.equals(PLAY)) {
            gameService.playNewGame();
        } else {
            System.out.println("ИГРА ЗАВЕРШЕНА");
        }

        scanner.close();
    }
}
```
```java
    public static void main() {
        GameService gameService = new GameService();
        MenuService menuService = new MenuService(gameService);

        menuService.startGame();
    }
```
****
10. Общее форматирование проекта.
Пожалуйста не забывай удалять пробелы там, где это не нужно и не забывай удалять лишние символы в самой программе.
А вот о чём конкретно я говорю:
- [тыц](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L139-L141)
- [тыц](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L63-L65)
- [тыц](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L129-L132)
- [тыц](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/GameService.java#L157-L159)
- [тыц](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/MenuService.java#L29-L32)
- [тыц](https://github.com/nadillustrator/HangmanGame/blob/c7612010edea28205d9d3cdffeb932f835e0dc2b/src/service/MenuService.java#L7-L9)
