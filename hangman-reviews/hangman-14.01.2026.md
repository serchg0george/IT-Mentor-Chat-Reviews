Ревью на проект Hangman для [@Glazov_A](https://t.me/zhukovsd_it_chat/53243/285691)

**Что выглядит неплохо**
-
1. В целом адекватный нейминг для переменных и методов
2. Есть обработка ошибок в методе `processError()`

**Что нужно улучшить:**
-
1. Игра не работает, у меня ни файла со словами, ни того адреса, по которому он должен располагаться.
На твоём месте я бы сюда добавил прямо в проект файл и считывал всё из него, а потом уже можно было бы добавить возможность 
подключать свои слова

<img width="2430" height="920" alt="image" src="https://github.com/user-attachments/assets/a203f436-9977-49b4-a8c2-db990886f193" />

2. Абсолютно вся логика содержится в одном классе, что нарушает Single Responsibility Principle.
Метод main() должен быть в классе Main.
Методы которые не отвечают за непосредственно логику игры должны быть вынесены в классы-хелперы, например следующим образом:

<img width="854" height="218" alt="image" src="https://github.com/user-attachments/assets/177bae9e-e572-49b3-974e-efc5af3ef4fb" />

Где классы `FileReader` и `BaseSprite` должны содержать методы ```readWords()``` и константы со спрайтами типа `STAGES_HANGMAN` соответственно

Логику самой игры можно вынести в отдельный класс, например `GameRunner` который будет содержать логику самого процесса игры и метод который будет вызывать её `run()`

<img width="2578" height="472" alt="image" src="https://github.com/user-attachments/assets/f165517a-1ff3-4361-982b-2cea516648c3" />

Методы `getRandomWord(), openLetter(), processError` я бы вынес в класс `GameService`

<img width="3866" height="654" alt="image" src="https://github.com/user-attachments/assets/3bd8ac14-de9f-4d2e-b934-b9d580abc763" />


3. В классе Main должна быть только точка входа в программу, больше ничего.
<img width="1748" height="502" alt="image" src="https://github.com/user-attachments/assets/0ae5d0d0-79d2-43f9-9aa2-69102b51be78" />

4. Логика игры трудно-читаема, слишком много всего в одном месте, вот как это выглядит в классе `GameRunner` (это твоя логика которая обёрнута в метод `run()`):
На твоём месте в первую очередь я бы эту логику разбил на методы поменьше, а после этого исходя из назначения методов их можно было бы распределеить по другим классам и здесь
оставить только действительное необходимые
```java
public class GameRunner {
    public static void run() throws IOException {

        List<String> words = readWords();

        try (Scanner sc = new Scanner(System.in)) {
            while (true) {

                int attemptCount = STAGES_HANGMAN.length;

                System.out.println("-----------------------------------");
                System.out.printf("Игра ВИСЕЛИЦА \nPush [N]ew game or [E]xit\n");
                System.out.println("-----------------------------------");

                String s = "";
                do {
                    s = sc.nextLine().toUpperCase();
                    if (s.equals("E")) {
                        System.out.println("Вы вышли из игры");
                        return;
                    } else if (!s.equals("N")) {
                        System.out.println("Вы вводите неверное значение!!!");
                    }
                } while (!s.equals("N"));

                String randomWord = getRandomWord(words);
                char[] masked = "*".repeat(randomWord.length()).toCharArray();
                System.out.println("Загаданное слово: " + new String(masked));

                System.out.println("У вас " + attemptCount + " попыток отгадать слово");

                List<Character> errors = new ArrayList<>();
                int[] stageIndex = { 0 };

                while (attemptCount > 0 && new String(masked).contains("*")) {
                    System.out.println("Введите букву: ");
                    String input = sc.nextLine().toUpperCase();

                    if (!input.matches("[А-ЯЁ]")) {
                        System.out.println("Введите ОДНУ РУССКУЮ букву");
                        continue;
                    }

                    char letter = input.charAt(0);

                    if (errors.contains(letter) || new String(masked).indexOf(letter) >= 0) {
                        System.out.println("Вы уже вводили эту букву");
                        continue;
                    }

                    boolean found = openLetter(randomWord, masked, letter);

                    System.out.println("Текущее слово: " + new String(masked));

                    if (!found) {

                        attemptCount = processError(errors, letter, attemptCount, stageIndex);
                        System.out.println("Текущее слово: " + new String(masked));

                        if (attemptCount == 0)
                            break;

                        String attempt = switch (attemptCount) {
                            case 1 -> "попытка";
                            case 2, 3, 4 -> "попытки";
                            default -> "попыток";
                        };

                        System.out.println("Осталось " + attemptCount + " " + attempt + " отгадать слово");

                    } else {
                        // понимаю что код задвоен(из-за перехода от 5 к 4, при attempt = 4, падеж был от 5), не смог реализовать иначе.
                        // switch выносил из блока if параллельно выносил счетчик из метода, но все равно работало некорректно и метод получался слепым
                        String attempt = switch (attemptCount) {
                            case 1 -> "попытка";
                            case 2, 3, 4 -> "попытки";
                            default -> "попыток";
                        };

                        System.out.println("Верно! Осталось " + attemptCount + " " + attempt + " отгадать слово");
                        System.out.println("ошибки: " + errors);
                    }
                }

                if (!new String(masked).contains("*")) {
                    System.out.println("-----------------------------------");
                    System.out.println("Поздравляем! Вы выиграли, загаданное слово: " + randomWord);
                } else {
                    System.out.println("-----------------------------------");
                    System.out.println("Вы проиграли, загаданное слово: " + randomWord);
                }
                System.out.println("Хотите сыграть еще? y/n");
                String answer = sc.nextLine().toUpperCase();
                if (answer.equals("Y")) {
                    continue;
                }
                System.out.println("Игра закончена!");
                return;
            }
        }
    }
}
```

5. В методе `getRandomWord()` оставлен комментарий, который не должен там быть
<img width="2292" height="554" alt="image" src="https://github.com/user-attachments/assets/78cde4ba-0f08-4456-83fd-75d6ee42dd95" />

6. Метод `readWords()` слишком громоздкий, как и в пункте 4 этот метод следовало бы разбить на методы поменьше
```java
public class FileReader {
    public static List<String> readWords() throws IOException {
        String inputFile = "words.txt";

        File file = new File("C:\\Users\\aglaz\\eclipse-workspace\\HangMan1\\words.txt");
        System.out.println(file.getCanonicalPath());

        List<String> words = new ArrayList<>();

        try (BufferedReader reader = new BufferedReader(new java.io.FileReader(inputFile))) {
            String line;
            int wordCount = 0;
            while ((line = reader.readLine()) != null) {
                String[] lineWords = line.split("\\s+");
                for (String word : lineWords) {
                    if (!word.matches("[А-Яа-яЁё]+")) {
                        continue;
                    }
                    words.add(word);
                    wordCount++;
                }
            }
            System.out.println(wordCount + " слов");
        } catch (IOException e) {
            System.err.println("Ошибка чтения файла: " + e.getMessage());
        }
        if (words.isEmpty()) {
            System.out.println("Файл не содержит слов.");
        }
        return words;
    }
}
```

7. Пожалуйста, когда коммитишь изменения давай осмысленные комментарии к комитам, например:
`changed constant value for hangman sprint` и т.п. Это делается для того, чтобы тебе было проще понять что именно ты тогда делал даже не глядя на код

8. На твоём месте я бы избегал использования ключевого слова `static` в данном проекте, а вместо этого использовал бы Dependency Injection везде, где только можно. 
Это значительно упрощает тестирование
------

Все неизвестные термины просьба загуглить и попробовать реализовать их в этом или следующем проекте
