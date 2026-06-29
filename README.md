<a id="ru"></a>
# Watermark Autoencoder v0.1

**Русский** | [English](#en)

Проект демонстрирует удаление синтетических водяных знаков с изображений с помощью автокодировщика, mask-aware U-Net и детерминированного метода Pattern-aware inverse removal.

Основная часть проекта построена вокруг нейросетевого подхода: на чистые изображения накладывается синтетический текстовый watermark, после чего модели обучаются восстанавливать исходное изображение. Дополнительно реализован Pattern-aware inverse removal - инженерный метод для случая, когда известны точный шаблон watermark, alpha-карта и формула наложения.

## Ноутбуки

| Язык | Файл |
|---|---|
| Русский | [`watermark_autoencoder_ru.ipynb`](watermark_autoencoder_ru.ipynb) |
| English | [`watermark_autoencoder_en.ipynb`](watermark_autoencoder_en.ipynb) |

## Цель проекта

Цель проекта - показать полный pipeline работы с водяными знаками на изображениях:

1. загрузка чистых изображений;
2. генерация синтетического watermark;
3. наложение watermark на изображения;
4. обучение базового автокодировщика;
5. обучение улучшенной mask-aware U-Net;
6. сравнение моделей на тестовой выборке;
7. проверка на настоящем watermark из датасета;
8. демонстрация Pattern-aware inverse removal для известного синтетического watermark.

## Датасет

Используется датасет из задания:

[`https://storage.yandexcloud.net/academy.ai/watermarked.zip`](https://storage.yandexcloud.net/academy.ai/watermarked.zip)

В проекте используются чистые изображения без водяных знаков из папок `train/no-watermark` и `valid/no-watermark`. На эти изображения программно накладывается синтетический watermark, после чего формируются пары:

```text
image with watermark -> clean image
```

Также в ноутбуке есть отдельная проверка на настоящих изображениях из папки `valid/watermark`.

## Watermark generation

Для наложения синтетического водяного знака используется центрированный текстовый watermark. Он автоматически подбирает масштаб так, чтобы полностью помещаться внутри изображения размером `128x128`.

Watermark задаётся как полупрозрачный текстовый шаблон. Для него дополнительно строятся:

- RGB-шаблон watermark;
- alpha-карта прозрачности;
- бинарная маска области watermark.

Эти данные используются для обучения mask-aware U-Net и для демонстрации Pattern-aware inverse removal.

## Архитектура проекта

Pipeline построен в три этапа:

```text
baseline autoencoder -> mask-aware U-Net -> pattern-aware inverse removal
```

### 1. Baseline autoencoder

Базовая модель - свёрточный автокодировщик. Он получает изображение с watermark и пытается восстановить чистое изображение.

Модель показывает, что простой автокодировщик способен уменьшать часть шума в области watermark, но часто повреждает фон и мелкие детали изображения.

### 2. Mask-aware U-Net

Mask-aware U-Net - улучшенная модель, которая обучается не только восстанавливать RGB-изображение, но и предсказывать alpha-карту watermark.

Такой подход позволяет:

- сильнее учитывать область watermark;
- защищать фон от лишних изменений;
- анализировать, где модель обнаружила watermark;
- лучше сохранять структуру изображения.

### 3. Pattern-aware inverse removal

Pattern-aware inverse removal - не нейросетевая модель, а детерминированный метод. Он используется как верхняя граница качества для синтетического watermark, когда известны:

- точный RGB-шаблон watermark;
- alpha-карта;
- формула смешивания watermark с исходным изображением.

Метод показывает почти полное удаление синтетического watermark, но напрямую не применим к произвольным реальным watermark без знания точного шаблона и прозрачности.

## Аугментации

В проекте используется больше пяти модификаций изображений:

- horizontal flip;
- brightness adjustment;
- contrast adjustment;
- saturation adjustment;
- Gaussian noise;
- rotation;
- blur;
- hue shift;
- gamma correction;
- sharpening.

Аугментации позволяют расширить обучающую выборку и сделать модели устойчивее к изменениям изображения.

## Пошаговое обучение

Обучение проводится поэтапно, чтобы экономить оперативную память и лучше подходить для среды Google Colab.

Данные разбиваются на несколько частей. На каждом шаге:

1. загружается часть изображений;
2. строятся обучающие пары;
3. модель дообучается;
4. временные массивы удаляются из памяти.

В ноутбуке используется не менее трёх шагов обучения.

## Пример результатов

Результаты могут немного отличаться при повторном запуске, так как обучение нейросетей зависит от стохастических факторов.

Пример результатов на синтетическом watermark:

| Подход | PSNR, dB | SSIM | Интерпретация |
|---|---:|---:|---|
| Baseline autoencoder | 23.2282 | 0.6730 | Удаляет часть watermark, но заметно повреждает фон |
| Mask-aware U-Net | 29.2349 | 0.9608 | Лучше сохраняет фон и улучшает качество изображения |
| Pattern-aware inverse removal | 64.7743 | 0.999945 | Почти полностью удаляет известный синтетический watermark |

Проверка на настоящем watermark из датасета:

| Подход | PSNR, dB | SSIM | Интерпретация |
|---|---:|---:|---|
| Baseline autoencoder | 20.9818 | 0.6971 | Сильно ухудшает изображение |
| Mask-aware U-Net | 37.8809 | 0.9900 | Почти не портит изображение, но не даёт заметного удаления настоящего watermark |

Результаты показывают, что mask-aware U-Net лучше обобщается и безопаснее работает с фоном, чем базовый автокодировщик. Однако настоящий watermark отличается от синтетического, поэтому для его уверенного удаления требуется отдельное обучение на соответствующих реальных парах.

## Демонстрации в ноутбуках

В ноутбуках представлены:

- исходные изображения;
- изображения с наложенным watermark;
- предсказания моделей;
- карты остаточного шума;
- alpha-карты watermark;
- сравнение метрик PSNR и SSIM;
- примеры работы на синтетическом watermark;
- примеры работы на настоящем watermark из датасета.

## Технологии

- Python
- TensorFlow / Keras
- NumPy
- pandas
- OpenCV
- Matplotlib
- scikit-image
- scikit-learn

## Как запустить

Откройте нужный ноутбук:

- `watermark_autoencoder_ru.ipynb` - русская версия;
- `watermark_autoencoder_en.ipynb` - английская версия.

Затем выполните все ячейки по порядку. Датасет загружается автоматически внутри ноутбука.

## Ограничения

Mask-aware U-Net обучается на синтетическом watermark, поэтому лучше всего работает с водяными знаками, похожими на обучающий шаблон.

Pattern-aware inverse removal показывает почти идеальный результат только для известного синтетического watermark. Для произвольного реального watermark без точного шаблона этот подход не гарантирует полного восстановления изображения.

---

<a id="en"></a>
# Watermark Autoencoder v0.1

[Русский](#ru) | **English**

This project demonstrates synthetic watermark removal from images using a convolutional autoencoder, a mask-aware U-Net, and a deterministic Pattern-aware inverse removal method.

The main part of the project is based on a neural-network approach: a synthetic text watermark is applied to clean images, and the models are trained to reconstruct the original clean image. In addition, the project includes Pattern-aware inverse removal - an engineering method for the case where the exact watermark pattern, alpha map, and blending formula are known.

## Notebooks

| Language | File |
|---|---|
| Russian | [`watermark_autoencoder_ru.ipynb`](watermark_autoencoder_ru.ipynb) |
| English | [`watermark_autoencoder_en.ipynb`](watermark_autoencoder_en.ipynb) |

## Project goal

The goal of the project is to demonstrate a complete image-watermark pipeline:

1. loading clean images;
2. generating a synthetic watermark;
3. applying the watermark to images;
4. training a baseline autoencoder;
5. training an improved mask-aware U-Net;
6. comparing models on the test set;
7. testing on real watermark images from the dataset;
8. demonstrating Pattern-aware inverse removal for a known synthetic watermark.

## Dataset

The project uses the dataset from the assignment:

[`https://storage.yandexcloud.net/academy.ai/watermarked.zip`](https://storage.yandexcloud.net/academy.ai/watermarked.zip)

The project uses clean images without watermarks from the `train/no-watermark` and `valid/no-watermark` folders. A synthetic watermark is applied programmatically, and the following training pairs are generated:

```text
image with watermark -> clean image
```

The notebook also includes a separate evaluation on real watermarked images from the `valid/watermark` folder.

## Watermark generation

A centered text watermark is used as the synthetic watermark. Its scale is selected automatically so that it fully fits inside a `128x128` image.

The watermark is represented as a semi-transparent text pattern. The following data are also generated:

- RGB watermark pattern;
- alpha transparency map;
- binary watermark-area mask.

These components are used for mask-aware U-Net training and for the Pattern-aware inverse removal demonstration.

## Project architecture

The pipeline consists of three stages:

```text
baseline autoencoder -> mask-aware U-Net -> pattern-aware inverse removal
```

### 1. Baseline autoencoder

The baseline model is a convolutional autoencoder. It receives a watermarked image and tries to reconstruct the clean image.

The model shows that a simple autoencoder can reduce part of the distortion in the watermark area, but it often damages the background and fine image details.

### 2. Mask-aware U-Net

Mask-aware U-Net is an improved model that learns not only to reconstruct the RGB image, but also to predict the watermark alpha map.

This approach makes it possible to:

- pay more attention to the watermark area;
- protect the background from unnecessary changes;
- analyze where the model detects the watermark;
- preserve image structure more accurately.

### 3. Pattern-aware inverse removal

Pattern-aware inverse removal is not a neural model. It is a deterministic method used as an upper quality bound for a synthetic watermark when the following are known:

- exact RGB watermark pattern;
- alpha map;
- formula used to blend the watermark with the original image.

The method demonstrates almost complete removal of the synthetic watermark, but it cannot be directly applied to arbitrary real watermarks without knowing the exact pattern and transparency.

## Augmentations

The project uses more than five image transformations:

- horizontal flip;
- brightness adjustment;
- contrast adjustment;
- saturation adjustment;
- Gaussian noise;
- rotation;
- blur;
- hue shift;
- gamma correction;
- sharpening.

Augmentations extend the training set and make the models more robust to image variations.

## Stepwise training

Training is performed step by step to reduce RAM usage and make the notebook more suitable for Google Colab.

The data are split into several parts. At each step:

1. a portion of images is loaded;
2. training pairs are built;
3. the model is trained further;
4. temporary arrays are removed from memory.

The notebook uses at least three training steps.

## Example results

The results may vary slightly between runs because neural-network training is stochastic.

Example results on the synthetic watermark:

| Approach | PSNR, dB | SSIM | Interpretation |
|---|---:|---:|---|
| Baseline autoencoder | 23.2282 | 0.6730 | Removes part of the watermark, but noticeably damages the background |
| Mask-aware U-Net | 29.2349 | 0.9608 | Preserves the background better and improves image quality |
| Pattern-aware inverse removal | 64.7743 | 0.999945 | Almost completely removes the known synthetic watermark |

Evaluation on real watermark images from the dataset:

| Approach | PSNR, dB | SSIM | Interpretation |
|---|---:|---:|---|
| Baseline autoencoder | 20.9818 | 0.6971 | Strongly degrades the image |
| Mask-aware U-Net | 37.8809 | 0.9900 | Almost does not damage the image, but does not noticeably remove the real watermark |

The results show that mask-aware U-Net generalizes better and works more safely with the background than the baseline autoencoder. However, the real watermark differs from the synthetic one, so reliable removal requires separate training on matching real pairs.

## Notebook demonstrations

The notebooks include:

- original images;
- images with synthetic watermark;
- model predictions;
- residual noise maps;
- watermark alpha maps;
- PSNR and SSIM metric comparison;
- examples on synthetic watermark;
- examples on real watermark images from the dataset.

## Technologies

- Python
- TensorFlow / Keras
- NumPy
- pandas
- OpenCV
- Matplotlib
- scikit-image
- scikit-learn

## How to run

Open the required notebook:

- `watermark_autoencoder_ru.ipynb` - Russian version;
- `watermark_autoencoder_en.ipynb` - English version.

Then run all cells in order. The dataset is downloaded automatically inside the notebook.

## Limitations

Mask-aware U-Net is trained on a synthetic watermark, so it works best with watermarks that are similar to the training pattern.

Pattern-aware inverse removal produces an almost perfect result only for a known synthetic watermark. For an arbitrary real watermark without the exact pattern, this approach does not guarantee full image reconstruction.
