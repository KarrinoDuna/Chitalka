# -*- coding: utf-8 -*-
# ========== НАСТРОЙКИ ==========
import os
import sys
import pygame
import asyncio
import edge_tts
from gtts import gTTS
from pydub import AudioSegment
from pydub.playback import play
import warnings

# 0 Измените на True чтобы сохранять файлы
SAVE_TEMP_FILES = False  # Измените на True чтобы сохранять файлы

# Игнорировать предупреждения pydub о ffmpeg
warnings.filterwarnings("ignore", message="Couldn't find ffmpeg")

# 1. Выбор движка (1-gTTS, 2-edge-tts, 3-pyttsx3(нужно доделать))
TTS_ENGINE = 2

# 2. Общие настройки
TEMPORARY_AUDIO_FILE = "temp_audio.mp3"
DEFAULT_ENCODINGS = ['utf-8', 'cp1251', 'windows-1251']

# 3. Настройки gTTS+pydub
G_TTS_LANG = 'ru'
G_TTS_SLOW = False  # Замедленная речь
G_TTS_SPEED = 1.0  # Скорость (1.0 - нормальная)

# 4. Настройки edge-tts

# ========== РУССКОЯЗЫЧНЫЕ ГОЛОСА EDGE TTS ==========
# Женские голоса:
# 'ru-RU-SvetlanaNeural'      - основной рекомендуемый женский голос
# 'ru-RU-DariyaNeural'        - мягкий эмоциональный женский
# 'ru-RU-IrinaNeural'         - четкий дикторский женский
# 'ru-RU-LarisaNeural'        - телевизионная подача

# Мужские голоса:
# 'ru-RU-DmitryNeural'        - глубокий мужской
# 'ru-RU-SvyatoslavNeural'    - мягкий повествовательный

# Специальные:
# 'ru-RU-FlorNeural'          - экспериментальный необычный тембр

# Для использования просто скопируйте нужное значение в переменную:
# EDGE_TTS_VOICE = 'ru-RU-SvetlanaNeural'  # <- пример выбора голоса
# ==================================================

EDGE_TTS_VOICE = 'ru-RU-SvetlanaNeural'  # Более стабильный голос
EDGE_TTS_RATE = '+100%'  # Скорость (-50% до +100%)
EDGE_TTS_VOLUME = '+0%'  # Громкость

# 5. Настройки pyttsx3 (если выбран)
PYTTSX3_RATE = 150  # Скорость речи
PYTTSX3_VOLUME = 0.9  # Громкость (0-1)
PYTTSX3_VOICE = None  # None - авто выбор, или укажите ID


# ========== РЕАЛИЗАЦИЯ ==========
def read_text_from_file(filename):
    """Чтение текста с автоопределением кодировки"""
    for encoding in DEFAULT_ENCODINGS:
        try:
            with open(filename, 'r', encoding=encoding) as file:
                return file.read()
        except (UnicodeDecodeError, LookupError):
            continue
    raise UnicodeDecodeError(f"Не удалось прочитать файл {filename}")


def play_audio(file_path):
    """Воспроизведение аудио через pygame"""
    try:
        pygame.mixer.init()
        pygame.mixer.music.load(file_path)
        pygame.mixer.music.play()

        while pygame.mixer.music.get_busy():
            pygame.time.Clock().tick(10)

    except Exception as e:
        print(f"Ошибка воспроизведения: {e}")
    finally:
        pygame.mixer.quit()


def speak_gtts(text):
    """Оффлайн синтез через gTTS с pydub"""
    try:
        # Явное указание пути к ffmpeg
        AudioSegment.converter = r"D:\Piton\ffmpeg\bin\ffmpeg.exe"
        AudioSegment.ffprobe = r"D:\Piton\ffmpeg\bin\ffprobe.exe"

        tts = gTTS(text=text, lang=G_TTS_LANG, slow=G_TTS_SLOW)
        tts.save(TEMPORARY_AUDIO_FILE)

        if G_TTS_SPEED != 1.0:
            audio = AudioSegment.from_mp3(TEMPORARY_AUDIO_FILE)
            audio = audio.speedup(playback_speed=G_TTS_SPEED)
            audio.export(TEMPORARY_AUDIO_FILE, format="mp3")

        play_audio(TEMPORARY_AUDIO_FILE)

    except Exception as e:
        print(f"Ошибка gTTS: {e}")
    finally:
        if not SAVE_TEMP_FILES and os.path.exists(TEMPORARY_AUDIO_FILE):
            try:
                os.remove(TEMPORARY_AUDIO_FILE)
            except:
                pass


async def async_edge_tts_save(text, output_file):
    """Асинхронный синтез через edge-tts"""
    try:
        communicate = edge_tts.Communicate(
            text=text,
            voice=EDGE_TTS_VOICE,
            rate=EDGE_TTS_RATE,
            volume=EDGE_TTS_VOLUME
        )
        await communicate.save(output_file)
        return True
    except Exception as e:
        print(f"Ошибка edge-tts: {str(e)}")
        return False


def speak_edge_tts(text):
    """Обертка для синхронного вызова edge-tts"""
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    try:
        success = loop.run_until_complete(
            async_edge_tts_save(text, TEMPORARY_AUDIO_FILE)
        )
        if success and os.path.exists(TEMPORARY_AUDIO_FILE):
            play_audio(TEMPORARY_AUDIO_FILE)
    except Exception as e:
        print(f"Ошибка: {e}")
    finally:
        loop.close()
        if not SAVE_TEMP_FILES and os.path.exists(TEMPORARY_AUDIO_FILE):
            try:
                os.remove(TEMPORARY_AUDIO_FILE)
            except:
                pass


def speak_pyttsx3(text):
    """Оффлайн синтез через pyttsx3"""
    try:
        import pyttsx3
        engine = pyttsx3.init()

        if PYTTSX3_RATE:
            engine.setProperty('rate', PYTTSX3_RATE)
        if PYTTSX3_VOLUME:
            engine.setProperty('volume', PYTTSX3_VOLUME)
        if PYTTSX3_VOICE:
            engine.setProperty('voice', PYTTSX3_VOICE)

        engine.say(text)
        engine.runAndWait()

    except Exception as e:
        print(f"Ошибка pyttsx3: {e}")


def speak_text(text):
    """Основная функция синтеза речи"""
    if TTS_ENGINE == 1:
        speak_gtts(text)
    elif TTS_ENGINE == 2:
        speak_edge_tts(text)
    elif TTS_ENGINE == 3:
        speak_pyttsx3(text)
    else:
        raise ValueError("Неверный выбор движка TTS")


if __name__ == "__main__":
    if len(sys.argv) > 1:
        filename = sys.argv[1]
    else:
        filename = "text.txt"

    try:
        text = read_text_from_file(filename)
        print(f"Читаю файл: {filename}...")
        speak_text(text)
        print("Готово!")
    except FileNotFoundError:
        print(f"Файл {filename} не найден!")
    except UnicodeDecodeError as e:
        print(f"Ошибка чтения файла: {e}")
    except Exception as e:
        print(f"Произошла ошибка: {e}")
