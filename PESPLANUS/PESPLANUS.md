# Описание проекта: Сервис для постановки диагноза плоскостопия по рентгеновским снимкам

Программа автоматизированного анализа цифровых рентгенограмм стоп предназначена для морфометрического анализа рентгенографических изображений стопы с целью поддержки принятия врачебных решений. Основной функционал программы включает автоматическое измерение углов и расстояний между костными структурами. Клиентское приложение предназначено для интеграции сервера обработки в информационный контур медицинской организации. 

## Что было сделано:
1. **Основная информация о плоскостопии, постановка задач**  
   - Перед началом работы проведено ознакомление с медицинской терминологией, определены основные задачи.
 
2. **Реализованные ML модели**
   - Для продольного плоскостопия была обучена модель на основе архитектуры Unet3+, до этого были испробованы Unet и Unet++, но наилучшее качество показала Unet3+. В качестве энкодера был выбран предобученный на датасете ImageNet - слой EfficientNetB7. Датасет для обучения состоял из 3345 изображений, из которых  1934 были рентгены продольного и 1411 "мусорных"/нецелевых снимков. Предварительно по разметке из 3 точек были созданы бинарные маски с треугольной областью для целевых снимков и пустые черные маски для "мусора". В связи с ограниченным набором данных активно была применена аугментация. Лучшее качество было достигнуто при 100 эпохах и с использованием косинусоидального затухания learning rate , что помогло выйти из локальных минимумов.
     
     ```
     - Epoch 95: dice_coef improved from 0.95286 to 0.95348, saving model to /home/a100/Projects/Mak/Pesplanus/logs/pespLong__20250317-162246/keras_max_dice.mode
     168/168 [==============================] - 235s 1s/step - loss: 0.0267 - dice_coef: 0.9535 - bce_dice_loss: 0.0267 - focal_loss: 0.0031 - val_loss: 0.0239 - val_dice_coef: 0.9583 - val_bce_dice_loss:
     0.0239 - val_focal_loss: 0.0028 - lr: 8.8564e-06
     ```
     На тестовых данных dice_coef = 0.94

     🔗 [КОД](https://github.com/makarstasia/Pesplanus/blob/main/mak_pesplanus_long.ipynb) 
   - Для поперечного плоскостопия было обучено 3 идентичных модели для сегментации костей, архитектура модели была взята аналогично продольному. Датасет для обучения состоял из 719 изображений, 369 из которых были целевыми и 350 "мусорными", так же использовалась аугментация и косинусоидальное затухания lr и 100 эпох. Перед обучением были созданы бинарные маски по размеченным точкам контура  для каждой из 3 костей (фаланга, 1 плюсневая, 2 плюсневая)

     ```
     Epoch 94: dice_coef improved from 0.96313 to 0.96435, saving model to /home/a100/Projects/Mak/Pesplanus/logs/pesp_mask_4_полная_20250314-082818/keras_max_dice.model
     36/36 [==============================] - 107s 3s/step - loss: 0.0190 - dice_coef: 0.9643 - log_cosh_dice_loss: 6.7872e-04 - bce_dice_loss: 0.0190 - focal_loss: 0.0010 - val_loss: 0.0250 -                     val_dice_coef:  0.9540 - val_log_cosh_dice_loss: 0.0011 - val_bce_dice_loss: 0.0250 - val_focal_loss: 0.0018 - lr: 1.2042e-05
     ```
     На тестовых данных dice_coef = 0.94

     🔗 [КОД](https://github.com/makarstasia/Pesplanus/blob/main/mak_pesplanus_mask2.ipynb)  
   - Для определения стороны ноги (правая/левая) была обучена YOLOv8 для детекции букв (R, L, П, Л). Датасет состоял из 4392 изображений, аннотированных bbox'ами
     ```
     Результаты на валидаионном наборе: 
     Class     Images  Instances      Box(P          R      mAP50  m
                   all       1183       1253      0.962      0.958      0.978      0.766
                     Л        445        446      0.983      0.987      0.994      0.844
                     П        448        448      0.975      0.987      0.993      0.857
                     R        200        200      0.961       0.94      0.967      0.682
                     L        159        159       0.93       0.92      0.958      0.683
     ```
     🔗 [КОД](https://github.com/makarstasia/yolo_letter_detect)
2. **Реализованние API**
   - Для высчитывания необходимых углов были разработаны соответствующие алгоритмы и собрана API, упакованная в дальшейшем в докер контейнер.
    🔗 [КОД](https://github.com/makarstasia/yolo_letter_detect)
3. **Интеграция**  
   - Для интеграции сервиса с паксом был создан ERIS клиент.
    🔗 [КОД](https://github.com/makarstasia/eris_v4.0_flat_feet)


**Визуальное представление сервиса:**

![Обработка поперечного плоскостопия](https://raw.githubusercontent.com/makarstasia/MY_PROJECTS/main/PESPLANUS/1.png)

![Обработка продольного плоскостопия](https://raw.githubusercontent.com/makarstasia/MY_PROJECTS/main/PESPLANUS/2.png)

![Обработка продольного плоскостопия](https://raw.githubusercontent.com/makarstasia/MY_PROJECTS/main/PESPLANUS/3.png)
   

