# RnOPC
Practical and laboratory work in the discipline of refactoring and optimization of program code

**Практическая работа 1.**
1 Длинные методы.
Методы, содержащие слишком много строк кода, сложно читать, тестировать и сопровождать. В контексте системы управления медиаконтентом, например, метод, отвечающий за публикацию контента, может содержать логику проверки доступа, обработки форматов файлов, подключения к хранилищу и отправки уведомлений.
Пример плохого кода:

public void publishContent(String contentId) {
    // Проверка доступа пользователя
    User user = userService.getCurrentUser();
    if (!user.hasPermission("PUBLISH")) {
        throw new SecurityException("Access denied");
    }

    // Загрузка контента
    Content content = contentRepository.findById(contentId);
    if (content == null) {
        throw new IllegalArgumentException("Content not found");
    }

    // Проверка формата файла
    if (!isValidFormat(content.getFile())) {
        throw new IllegalArgumentException("Invalid file format");
    }

    // Публикация контента
    content.setPublished(true);
    contentRepository.save(content);

    // Отправка уведомлений
    notificationService.notifyUsers("Content published", content.getTitle());
}

Для решения данной проблемы необходимо разделить метод на несколько мелких методов, каждый из которых выполняет одну задачу.
Пример исправленного кода:

public void publishContent(String contentId) {
    validateUserPermissions();
    Content content = loadContent(contentId);
    validateContentFormat(content);
    savePublishedContent(content);
    sendPublicationNotification(content);
}

private void validateUserPermissions() {
    User user = userService.getCurrentUser();
    if (!user.hasPermission("PUBLISH")) {
        throw new SecurityException("Access denied");
    }
}

private Content loadContent(String contentId) {
    Content content = contentRepository.findById(contentId);
    if (content == null) {
        throw new IllegalArgumentException("Content not found");
    }
    return content;
}

private void validateContentFormat(Content content) {
    if (!isValidFormat(content.getFile())) {
        throw new IllegalArgumentException("Invalid file format");
    }
}

private void savePublishedContent(Content content) {
    content.setPublished(true);
    contentRepository.save(content);
}

private void sendPublicationNotification(Content content) {
    notificationService.notifyUsers("Content published", content.getTitle());
}

В результате логика разделена на несколько независимых блоков. Улучшается читаемость, тестируемость

2 Дублирование кода
Повторение одинаковых блоков кода затрудняет сопровождение и увеличивает вероятность ошибок. Например, проверка прав доступа может использоваться в разных местах системы.
Пример плохого кода:

public void publishContent(String contentId) {
    User user = userService.getCurrentUser();
    if (!user.hasPermission("PUBLISH")) {
        throw new SecurityException("Access denied");
    }
    // Остальная логика
}

public void archiveContent(String contentId) {
    User user = userService.getCurrentUser();
    if (!user.hasPermission("ARCHIVE")) {
        throw new SecurityException("Access denied");
    }
    // Остальная логика
}

Для решения надо вынести общий код в отдельный метод или класс.
Пример исправленного кода:

public void publishContent(String contentId) {
    validateUserPermission("PUBLISH");
    // Остальная логика
}

public void archiveContent(String contentId) {
    validateUserPermission("ARCHIVE");
    // Остальная логика
}

private void validateUserPermission(String permission) {
    User user = userService.getCurrentUser();
    if (!user.hasPermission(permission)) {
        throw new SecurityException("Access denied");
    }
}

В результате уменьшается количество кода, а также централизованная проверка прав упрощает изменение логики в будущем.

3 Плохие имена функций
Непонятные или неточные названия методов затрудняют понимание кода. Например, метод с именем process вряд ли объяснит, что он делает.
Пример некачественного кода:

public void process(String id) {
    // Логика обработки контента
}

	Вместо этого необходимо использовать название которое отражает суть метода.

public void publishContentById(String contentId) {
    // Логика публикации контента
}

В результате чего улучшается понимание кода.

4 Избыточные временные переменные
Использование лишних переменных увеличивает сложность и читаемость кода.
Пример плохого кода:

public String getContentTitle(Content content) {
    String title = content.getTitle();
    return title;
}

	Лучше сократить код, если переменная не сильно нужна.

public String getContentTitle(Content content) {
    return content.getTitle();
}

	В результате код становится более лаконичным.

5 Утечка памяти
Неправильное управление ресурсами, например, открытие потоков и их несвоевременное закрытие, приводит к утечке памяти.
Пример некачественного кода:

public void saveContentToFile(String content, String filePath) throws IOException {
    FileWriter writer = new FileWriter(filePath);
    writer.write(content);
}
	Используйте try-with-resources для автоматического освобождения ресурсов.
	Пример хорошего кода:

public void saveContentToFile(String content, String filePath) throws IOException {
    try (FileWriter writer = new FileWriter(filePath)) {
        writer.write(content);
    }
}

	В результате устраняется утечка памяти.

6 Длительное выполнение операций
Продолжительные операции, такие как обработка больших данных, могут замедлить систему.
Пример плохого кода:

public void processAllContents(List<Content> contents) {
    for (Content content : contents) {
        // Длительная обработка
    }
}

	Применим многопоточность или отложенную отработку.
	Реузультат:

	public void processAllContents(List<Content> contents) {
    contents.parallelStream().forEach(content -> {
        // Длительная обработка
    });
}

7 Сложная и запутанная структура классов
Классы с избыточной логикой или выполняющие сразу несколько задач становятся сложными для сопровождения и тестирования. Например, класс ContentManager может одновременно обрабатывать загрузку файлов, проверку прав и отправку уведомлений, что нарушает принцип единственной ответственности (SRP).
Пример плохого кода:

public class ContentManager {
    public void handleContent(String contentId) {
        // Проверка прав
        User user = userService.getCurrentUser();
        if (!user.hasPermission("MANAGE")) {
            throw new SecurityException("Access denied");
        }

        // Обработка контента
        Content content = contentRepository.findById(contentId);
        content.setStatus("Processed");

        // Уведомление
        notificationService.notifyUsers("Content processed", content.getTitle());
    }
}

Необходимо разделить ответственность между классами. Например использовать PermissionValidator, ContentProcessor и NotificationService.
Исправленный код:

public class ContentManager {
    private PermissionValidator permissionValidator;
    private ContentProcessor contentProcessor;
    private NotificationService notificationService;

    public ContentManager(PermissionValidator permissionValidator, 
                          ContentProcessor contentProcessor, 
                          NotificationService notificationService) {
        this.permissionValidator = permissionValidator;
        this.contentProcessor = contentProcessor;
        this.notificationService = notificationService;
    }

    public void handleContent(String contentId) {
        permissionValidator.validatePermission("MANAGE");
        Content content = contentProcessor.processContent(contentId);
        notificationService.notifyUsers("Content processed", content.getTitle());
    }
}

**Практическая работа 2**


![image](https://github.com/user-attachments/assets/cefbfbc5-4dc4-4ea7-828f-3e5e757da10d)
Рисунок 1 – Диаграмма user flow для роли «Маркетолог»

![image](https://github.com/user-attachments/assets/52cd4116-8ce6-4eff-9e21-dc23e4d40af6)
Рисунок 2 – Диаграмма user flow для роли «Администратор»

![image](https://github.com/user-attachments/assets/7bffe265-f67b-4ef8-8edd-7e294375d6cd)
Рисунок 3 – Диаграмма user flow для роли «Аналитик»

![image](https://github.com/user-attachments/assets/1c086dab-f477-4b80-be20-d8d795eebb70)
Рисунок 4 – Диаграмма user flow для роли «Пользователь



