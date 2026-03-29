# HW10-11 – компьютерное зрение в PyTorch: CNN, transfer learning, detection/segmentation

## 1. Кратко: что сделано

- **Часть A**: выбран датасет **STL10** (10 классов, изображения 96×96).  
- **Часть B**: выбран датасет **Pascal VOC 2012** и трек **detection** (Faster R-CNN ResNet50-FPN).  
- В части A сравнивались 4 модели: простая CNN без и с аугментациями (C1/C2), ResNet18 с замороженным backbone (C3) и с частичным fine-tuning (C4).  
- Во второй части сравнивались два режима фильтрации предсказаний: score_threshold = 0.3 (V1) и score_threshold = 0.7 (V2).

## 2. Среда и воспроизводимость

- **Python**: 3.10 (Colab)  
- **torch / torchvision**: 2.4+ / 0.19+  
- **Устройство**: GPU (T4)  
- **Seed**: 17  
- **Как запустить**: открыть `HW10-11.ipynb` и выполнить **Run All**.

## 3. Данные

### 3.1. Часть A: классификация

- **Датасет**: `STL10`  
- **Разделение**: train (4000) / val (1000) / test (8000). Разделение train/val выполнено 80/20.  
- **Базовые transforms**: `transforms.ToTensor()`  
- **Augmentation transforms**: RandomHorizontalFlip, ColorJitter, RandomRotation, RandomResizedCrop.  
- **Комментарий**: STL10 содержит 10 классов естественных объектов. Изображения имеют размер 96×96. Датасет относительно небольшой, поэтому важную роль играет регуляризация и перенос знаний из предобученных моделей.

### 3.2. Часть B: structured vision

- **Датасет**: `Pascal VOC 2012`  
- **Трек**: `detection`  
- **Что считается ground truth**: bounding boxes и классы объектов (20 классов).  
- **Какие предсказания использовались**: bounding boxes и confidence scores от pretrained Faster R-CNN ResNet50-FPN после фильтрации по score_threshold.  
- **Комментарий**: Pascal VOC — стандартный датасет для задач object detection. Он содержит разнообразные сцены с несколькими объектами, что позволяет адекватно оценивать качество локализации и классификации одновременно.

## 4. Часть A: модели и обучение (C1-C4)

- **C1 (simple-cnn-base)**: простая CNN (3 conv-блока + 2 fc-слоя) без аугментаций.  
- **C2 (simple-cnn-aug)**: та же архитектура с применением аугментаций.  
- **C3 (resnet18-head-only)**: pretrained ResNet18 с замороженным backbone, обучается только классификационная голова.  
- **C4 (resnet18-finetune)**: pretrained ResNet18 с размороженными layer4 + fc-слоем.

**Дополнительно**:
- **Loss**: CrossEntropyLoss  
- **Optimizer**: Adam  
- **Batch size**: 64  
- **Epochs**: 10  
- **Критерий выбора лучшей модели**: максимальная val_accuracy

## 5. Часть B: постановка задачи и режимы оценки (V1-V2)

- **Модель**: Faster R-CNN ResNet50-FPN (pretrained на COCO)  
- **V1**: `score_threshold = 0.3`  
- **V2**: `score_threshold = 0.7`  
- **Как считался IoU**: Intersection over Union с порогом 0.5 для определения истинно-положительных предсказаний.  
- **Как считались precision / recall**: на основе TP/FP/FN после применения score_threshold.

## 6. Результаты

**Ссылки на файлы**:
- Таблица результатов: [`./artifacts/runs.csv`](artifacts/runs.csv)  
- Лучшая модель части A: [`./artifacts/best_classifier.pt`](artifacts/best_classifier.pt)  
- Кривые обучения лучшей модели: [`./artifacts/figures/classification_curves_best.png`](artifacts/figures/classification_curves_best.png)  
- Сравнение C1–C4: [`./artifacts/figures/classification_compare.png`](artifacts/figures/classification_compare.png)  
- Визуализация аугментаций: [`./artifacts/figures/augmentations_preview.png`](artifacts/figures/augmentations_preview.png)  
- Метрики detection: [`./artifacts/figures/detection_metrics.png`](artifacts/figures/detection_metrics.png)  
- Примеры предсказаний: [`./artifacts/figures/detection_examples.png`](artifacts/figures/detection_examples.png)
- Конфигурация лучшей модели: [`./artifacts/best_classifier_config.json`](artifacts/best_classifier_config.json)

**Короткая сводка**:
- Лучший результат в части A показал эксперимент **C4** (ResNet18 с частичным fine-tuning) — val_accuracy = **0.868**, test_accuracy = **0.864**.  
- Аугментации не дали прироста (C2: 0.581 против C1: 0.591).  
- Transfer learning сильно улучшил качество: C3 (0.802) и особенно C4 значительно превзошли простые CNN.  
- Частичный fine-tuning (C4) оказался эффективнее, чем обучение только головы (C3).  
- В части B режим V1 (threshold 0.3) дал recall = 1.0 и precision = 0.318.  
- Режим V2 (threshold 0.7) повысил precision до 0.5, снизил recall до 0.571 и немного улучшил mean IoU (0.821 против 0.796).

## 7. Анализ

Простая CNN показала ограниченное качество (~59% val_accuracy), что объясняется небольшой глубиной сети и ограниченным размером обучающей выборки STL10. Аугментации в данном случае не принесли улучшения, возможно, из-за слишком агрессивных трансформаций.

Предобученная ResNet18 значительно повысила точность даже при замороженном backbone. Дополнительное обучение layer4 вместе с головой позволило модели лучше адаптироваться к особенностям датасета и дало заметный прирост качества.

В задаче detection изменение score_threshold наглядно демонстрирует классический trade-off между precision и recall. Более высокий порог отсекает слабые предсказания, повышая точность, но пропуская часть объектов. Выбранные метрики (precision, recall и mean IoU) хорошо отражают как способность модели находить объекты, так и качество их локализации.

Наиболее частые ошибки в detection — ложные срабатывания на сложном фоне при низком пороге и пропуски небольших или частично перекрытых объектов.

## 8. Итоговый вывод

Наиболее эффективным конфигурацией для классификации оказался **C4** — ResNet18 с частичным fine-tuning последних слоёв.

Основной вывод по transfer learning: даже относительно небольшое дообучение (layer4 + голова) даёт существенный прирост по сравнению с обучением только классификационной головы.

В задачах object detection важно понимать влияние confidence threshold на баланс precision и recall. Метрики IoU и mean IoU позволяют отдельно оценивать качество локализации объектов.

## 9. Приложение

- Полная таблица экспериментов — `runs.csv`  
- Графики обучения лучшей модели — `classification_curves_best.png`  
- Сравнение всех экспериментов части A — `classification_compare.png`  
- Примеры работы детектора с разными порогами — `detection_examples.png`