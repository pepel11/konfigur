## Этап 1. Минимальный прототип с конфигурацией
### Цель
Cоздать минимальное CLI-приложение и сделать его настраиваемым.
### Требование
1. Источником настраиваемых пользователем параметров является
конфигурационный файл формата YAML.
2. К настраиваемым параметрам относятся:
– Имя анализируемого пакета.
– URL-адрес репозитория или путь к файлу тестового репозитория.
– Режим работы с тестовым репозиторием.
– Версия пакета.
– Имя сгенерированного файла с изображением графа.
– Режим вывода зависимостей в формате ASCII-дерева.
3. (только для этого этапа) При запуске приложения вывести все параметры,
настраиваемые пользователем, в формате ключ-значение.
4. Реализовать и продемонстрировать обработку ошибок для всех параметров.
5. Результат выполнения этапа сохранить в репозиторий стандартно
оформленным коммитом.
 сделай на питоне
### Результат запуска
<img width="966" height="437" alt="image" src="https://github.com/user-attachments/assets/be5798d3-8bf2-4c61-bdca-788fa990ede6" />

### Код программы
```import yaml
import os
import sys
from typing import Dict, Any


class ConfigError(Exception):
    """Кастомное исключение для ошибок конфигурации"""
    pass


class DependencyAnalyzer:
    def __init__(self, config_path: str = "config.yaml"):
        self.config_path = config_path
        self.config = {}

    def load_config(self) -> Dict[str, Any]:
        """Загрузка конфигурации из YAML файла"""
        try:
            if not os.path.exists(self.config_path):
                raise ConfigError(f"Конфигурационный файл {self.config_path} не найден")

            with open(self.config_path, 'r', encoding='utf-8') as file:
                config = yaml.safe_load(file)

            if config is None:
                raise ConfigError("Конфигурационный файл пуст или имеет неверный формат")

            return config

        except yaml.YAMLError as e:
            raise ConfigError(f"Ошибка парсинга YAML: {e}")
        except IOError as e:
            raise ConfigError(f"Ошибка чтения файла: {e}")

    def validate_config(self, config: Dict[str, Any]) -> None:
        """Валидация параметров конфигурации"""
        required_fields = [
            'package_name',
            'repository_url',
            'test_repo_mode',
            'package_version',
            'output_image',
            'ascii_tree_output'
        ]

        # Проверка обязательных полей
        for field in required_fields:
            if field not in config:
                raise ConfigError(f"Обязательный параметр '{field}' отсутствует в конфигурации")

        # Валидация package_name
        if not isinstance(config['package_name'], str) or not config['package_name'].strip():
            raise ConfigError("Параметр 'package_name' должен быть непустой строкой")

        # Валидация repository_url
        if not isinstance(config['repository_url'], str) or not config['repository_url'].strip():
            raise ConfigError("Параметр 'repository_url' должен быть непустой строкой")

        # Валидация test_repo_mode
        valid_modes = ['local', 'remote']
        if config['test_repo_mode'] not in valid_modes:
            raise ConfigError(f"Параметр 'test_repo_mode' должен быть одним из: {valid_modes}")

        # Валидация package_version
        if not isinstance(config['package_version'], str) or not config['package_version'].strip():
            raise ConfigError("Параметр 'package_version' должен быть непустой строкой")

        # Валидация output_image
        if not isinstance(config['output_image'], str) or not config['output_image'].strip():
            raise ConfigError("Параметр 'output_image' должен быть непустой строкой")

        # Валидация ascii_tree_output
        if not isinstance(config['ascii_tree_output'], bool):
            raise ConfigError("Параметр 'ascii_tree_output' должен быть булевым значением")

    def display_config(self) -> None:
        """Отображение всех параметров конфигурации"""
        print("=" * 50)
        print("НАСТРОЙКИ ПРИЛОЖЕНИЯ")
        print("=" * 50)

        for key, value in self.config.items():
            print(f"{key}: {value}")

        print("=" * 50)

    def run(self) -> None:
        """Основной метод запуска приложения"""
        try:
            # Загрузка конфигурации
            self.config = self.load_config()

            # Валидация конфигурации
            self.validate_config(self.config)

            # Отображение параметров
            self.display_config()

            print("Приложение успешно запущено с указанной конфигурацией!")

        except ConfigError as e:
            print(f"Ошибка конфигурации: {e}")
            sys.exit(1)
        except Exception as e:
            print(f"Неожиданная ошибка: {e}")
            sys.exit(1)


def create_sample_config() -> None:
    """Создание примера конфигурационного файла"""
    sample_config = {
        'package_name': 'example-package',
        'repository_url': 'https://github.com/example/repo.git',
        'test_repo_mode': 'remote',
        'package_version': '1.0.0',
        'output_image': 'dependency_graph.png',
        'ascii_tree_output': True
    }

    with open('../../AppData/Roaming/JetBrains/PyCharmCE2024.2/scratches/config.yaml', 'w', encoding='utf-8') as file:
        yaml.dump(sample_config, file, default_flow_style=False, allow_unicode=True)

    print("Создан пример конфигурационного файла 'config.yaml'")


def main():
    """Точка входа в приложение"""
    if not os.path.exists('../../AppData/Roaming/JetBrains/PyCharmCE2024.2/scratches/config.yaml'):
        print("Конфигурационный файл не найден. Создаю пример...")
        create_sample_config()
        print("Отредактируйте config.yaml и запустите приложение снова")
        return

    # Запуск анализатора
    analyzer = DependencyAnalyzer()
    analyzer.run()


if __name__ == "__main__":
    main()```
