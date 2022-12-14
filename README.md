# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #5 выполнил(а):
- Амосова Варвара Ивановна
- РИ - 210940
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Интегрировать экономическую систему в проект Unity и обучить ML-Agent.

## Задание 1
### Измените параметры файла. yaml-агента и определить какие параметры и как влияют на обучение модели.

Ход работы:

- В начале работы необходимо открыть проект в Unity, представленный в методических указаниях. 
- Установка ML-агента на более позднюю версию:

![2022-12-02_17-45-32](https://user-images.githubusercontent.com/114309754/205367186-e1a22d90-2ba7-40bf-b503-205542adc8e2.png)

- С помощью Anaconda Prompt нужно активировать ML-агент и скачать библиотеки mlagents 0.28.0 и torch 1.7.1. 

```py
conda create -n MLAgents python=3.6
conda activate MLAgents
```

![2022-12-02_17-51-03](https://user-images.githubusercontent.com/114309754/205367434-8d693393-acfc-4115-8f3b-d30b05236dc9.png)
![2022-12-02_17-50-10](https://user-images.githubusercontent.com/114309754/205367435-b9b46209-4119-4142-a6d0-09454044c698.png)


```py
pip install mlagents==0.28.0
```
![2022-12-02_17-52-15](https://user-images.githubusercontent.com/114309754/205367888-5d0f868b-3df8-47db-9223-d8101b1c26f0.png)

```py
pip install torch~=1.7.1 -f https://download.pytorch.org/whl/torch_stable.html
```
![2022-12-02_18-47-41](https://user-images.githubusercontent.com/114309754/205367844-8ff41eec-586e-47df-ae1c-e23d6d08dfb5.png)

- Переходим к обучению модели.

Файл Move.cs:

```py
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class Move : Agent
{
[SerializeField] private GameObject goldMine;
[SerializeField] private GameObject village;
private float speedMove;
private float timeMining;
private float month;
private bool checkMiningStart = false;
private bool checkMiningFinish = false;
private bool checkStartMonth = false;
private bool setSensor = true;
private float amountGold;
private float pickaxeСost;
private float profitPercentage;
private float[] pricesMonth = new float[2];
private float priceMonth;
private float tempInf;

// Start is called before the first frame update
public override void OnEpisodeBegin()
{
// If the Agent fell, zero its momentum
if (this.transform.localPosition != village.transform.localPosition)
{
this.transform.localPosition = village.transform.localPosition;
}
checkMiningStart = false;
checkMiningFinish = false;
checkStartMonth = false;
setSensor = true;
priceMonth = 0.0f;
pricesMonth[0] = 0.0f;
pricesMonth[1] = 0.0f;
tempInf = 0.0f;
month = 1;
}
public override void CollectObservations(VectorSensor sensor)
{
sensor.AddObservation(speedMove);
sensor.AddObservation(timeMining);
sensor.AddObservation(amountGold);
sensor.AddObservation(pickaxeСost);
sensor.AddObservation(profitPercentage);
}

public override void OnActionReceived(ActionBuffers actionBuffers)
{
if (month < 3 || setSensor == true)
{
speedMove = Mathf.Clamp(actionBuffers.ContinuousActions[0], 1f, 10f);
Debug.Log("SpeedMove: " + speedMove);
timeMining = Mathf.Clamp(actionBuffers.ContinuousActions[1], 1f, 10f);
Debug.Log("timeMining: " + timeMining);
setSensor = false;
if (checkStartMonth == false)
{
Debug.Log("Start Coroutine StartMonth");
StartCoroutine(StartMonth());
}

if (transform.position != goldMine.transform.position & checkMiningFinish == false)
{
transform.position = Vector3.MoveTowards(transform.position, goldMine.transform.position, Time.deltaTime * speedMove);
}

if (transform.position == goldMine.transform.position & checkMiningStart == false)
{
Debug.Log("Start Coroutine StartGoldMine");
StartCoroutine(StartGoldMine());
}

if (transform.position != village.transform.position & checkMiningFinish == true)
{
transform.position = Vector3.MoveTowards(transform.position, village.transform.position, Time.deltaTime * speedMove);
}

if (transform.position == village.transform.position & checkMiningStart == true)
{
checkMiningFinish = false;
checkMiningStart = false;
setSensor = true;
amountGold = Mathf.Clamp(actionBuffers.ContinuousActions[2], 1f, 10f);
Debug.Log("amountGold: " + amountGold);
pickaxeСost = Mathf.Clamp(actionBuffers.ContinuousActions[3], 100f, 1000f);
Debug.Log("pickaxeСost: " + pickaxeСost);
profitPercentage = Mathf.Clamp(actionBuffers.ContinuousActions[4], 0.1f, 0.5f);
Debug.Log("profitPercentage: " + profitPercentage);

if (month != 2)
{
priceMonth = pricesMonth[0] + ((pickaxeСost + pickaxeСost * profitPercentage) / amountGold);
pricesMonth[0] = priceMonth;
Debug.Log("priceMonth: " + priceMonth);
}
if (month == 2)
{
priceMonth = pricesMonth[1] + ((pickaxeСost + pickaxeСost * profitPercentage) / amountGold);
pricesMonth[1] = priceMonth;
Debug.Log("priceMonth: " + priceMonth);
}

}
}
else
{
tempInf = ((pricesMonth[1] - pricesMonth[0]) / pricesMonth[0]) * 100;
if (tempInf <= 6f)
{
SetReward(1.0f);
Debug.Log("True");
Debug.Log("tempInf: " + tempInf);
EndEpisode();
}
else
{
SetReward(-1.0f);
Debug.Log("False");
Debug.Log("tempInf: " + tempInf);
EndEpisode();
}
}
}

IEnumerator StartGoldMine()
{
checkMiningStart = true;
yield return new WaitForSeconds(timeMining);
Debug.Log("Mining Finish");
checkMiningFinish = true;
}

IEnumerator StartMonth()
{
checkStartMonth = true;
yield return new WaitForSeconds(60);
checkStartMonth = false;
month++;

}
}
```

- Обучим модель и с помощью графиков и посмотрим на результат обучения:


```py
mlagents-learn Economic.yaml --run-id=Economic –-force
```

![2Sc0KkEEbJ8](https://user-images.githubusercontent.com/114309754/205368228-7bba10d6-6849-44b0-9058-4281dc494fce.jpg)

Установим TensorBoard для оценки результатов обучения:
```py
pip install tensorflow
```
![2022-12-02_19-38-13](https://user-images.githubusercontent.com/114309754/205368368-dfe406d5-3c2f-4eec-9f28-05f6a248c11e.png)
![2022-12-02_19-38-22](https://user-images.githubusercontent.com/114309754/205368371-6cd1dd2e-10ac-4f89-b99a-e5f7fd97c8c5.png)

Содержимое файла Economic.yaml:

```py
behaviors:
Economic:
trainer_type: ppo
hyperparameters:
batch_size: 1024
buffer_size: 10240
learning_rate: 1.0e-4
learning_rate_schedule: linear
beta: 1.0e-2
epsilon: 0.2
lambd: 0.95
num_epoch: 3
network_settings:
normalize: false
hidden_units: 128
num_layers: 2
reward_signals:
extrinsic:
gamma: 0.99
strength: 1.0
checkpoint_interval: 500000
max_steps: 750000
time_horizon: 64
summary_freq: 5000
self_play:
save_steps: 20000
team_change: 100000
swap_steps: 10000
play_against_latest_model_ratio: 0.5
window: 10
```
- После установки TensorBoard появились следующие графики:

![lY31s5Rk6SA](https://user-images.githubusercontent.com/114309754/205369346-c3f6a370-4242-47d7-a930-6d7f1036a179.jpg)

В следующих запусках меняем значение одного из параметров для того, чтобы проверить, как каждый параметр будет влиять на обучение модели.

- Изменим параметр batch_size на 2048.

![prdguRiXZvk](https://user-images.githubusercontent.com/114309754/205369882-f5168797-416e-4b52-9ff2-8efcd109b792.jpg)

- Изменим параметр beta на 1.5e-2.
.
![t2Wdrg1iasw](https://user-images.githubusercontent.com/114309754/205370087-ec2c222b-4629-4d7a-a0ee-979e95bf299b.jpg)

- Изменим параметры lambd: 0.9 и beta на 1.5e-2.

![b_-yW7y3G7Y](https://user-images.githubusercontent.com/114309754/205370815-b08abf02-a597-4957-bac9-65d8f87b46ed.jpg)

- Изменим параметр gamma на 0.9.

![UrjmzzSym0I](https://user-images.githubusercontent.com/114309754/205371206-f4f57a8c-1246-4184-a56a-7e9a77549d56.jpg)

- Изменим параметр num_epoch: 10

![VN-LlPZT0Q8](https://user-images.githubusercontent.com/114309754/205371194-df96e2f7-8848-45f8-9ef6-ead60ba67e68.jpg)

## Задание 2
### Опишите результаты, выведенные в TensorBoard.

- Cumulative Reward: среднее совокупное вознаграждение за эпизод по всем агентам. Должно увеличиваться во время успешной тренировки. О значении на каждой тренировке было сказано в задании 1.

- Episode Length: Средняя продолжительность каждого эпизода в окружающей среде для всех агентов. 

- Policy Loss: Средняя величина функции потерь. Соответствует тому, насколько сильно меняется политика (процесс принятия решений о действиях). Этот график должен уменьшаться во время успешной тренировки.

- Value Loss: Средняя потеря функции обновления значения. Соответствует тому, насколько хорошо модель способна предсказать значение каждого состояния. Этот график должен увеличиваться, пока агент учится, а затем уменьшаться, как только вознаграждение стабилизируется. 

- Графики в разделе Policy показывают изменение некоторых параметров, которые указаны в .yaml файле

- Beta: Идет монотонно вниз.

- Entropy: Показывает насколько случайны решения модели. Должен медленно уменьшаться во время успешного тренировочного процесса. 

- Epsilon: уменьшается (графики имеют разные начальные значения).

- Extrinsic Reward: Этот график соответствует среднему совокупному вознаграждению, полученному от окружающей среды за эпизод. 

- Extrinsic Value Estimate: Оценка среднего значения для всех положений, посещенных агентом. Должно увеличиваться во время успешной тренировки. 

- Learning Rate: все графики идут вниз.

С помощью графиков можно понять, какие параметры стоит изменить или оставить для лучшей тренировки модели.


## Выводы
В ходе выполнения данной лабораторной работы, я научился интегрировать экономическую систему в проект Unity в связке с MLAgent, наблюдать за изменениями при изменении параметров в Yaml файле, делать графики на основе результатов обучения MLAgent и анализировать графики.



| Plugin | README |
| ------ | ------ |
| GitHub | [plugins/github/README.md][PlGh] |
| Unity | [plugins/unity/README.md][PlU] |
| Anaconda Prompt | [plugins/anacondaprompt/README.md][PlAP] |
| TensorBoard | [plugins/tensorboard/README.md][PlTB] |
| Visual Studio Code | [plugins/visualstudiocode/README.md][PlVSC] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
