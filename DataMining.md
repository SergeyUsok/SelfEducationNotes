# Data Mining

## The Ancient Art of the Numerati (Data Mining) [http://guidetodatamining.com/](http://guidetodatamining.com/)

**1. Создаем систему рекомендаций, основанную на оценках пользователей**

**(User-based).**

Допустим, есть таблица с пользователями, песнями и их оценками (от 1 до 5):

|   | Unforgiven | Paradise | Ghost | Sniper |
| --- | --- | --- | --- | --- |
| Ben | 1.5 |   |   | 4 |
| Tom | 3 | 5 |   |   |
| Ron | 3.5 |   | 2 | 4.5 |
| Sam |   | 3 | 5 |   |

Алгоритмы:

1. Minkowski Distance Metric

 ![](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/DataMining/Minkowski.png)

- Euclidean Distance when r=2

- Manhattan Distance when r=1

```csharp
public static double MinkowskiDistance(User target, User neighbor, double r)

{
    var combinedBySongs = target.Songs.Join(neighbor.Songs,
                                            s => s.Name, s => s.Name,
                                            (t, n) => new { TargetRating = t.Rating, NeighborRating = n.Rating });

    var sum = combinedBySongs.Sum(p => Math.Pow(Math.Abs(p.TargetRating - p.NeighborRating), r));

    returnMath.Pow(sum, 1/r);
}
```

4. Cosine Similarity

 ![](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/DataMining/Cosine.png) где ||x|| это ![](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/DataMining/Cosine_part2.png)

3. Pearson Correlation Coefficient

 ![](https://github.com/SergeyUsok/SelfEducationNotes/blob/master/img/DataMining/PearsonCorrelation.png)

Результат от -1 до 1, где 1 прямая корреляция и схожесть

Реализованный алгоритм ниже:

```csharp
        public static double PearsonCorrelation(User target, User neighbor)
        {
            var combined = target.Songs.Join(neighbor.Songs,
                                             s => s.Name, s => s.Name,
                                             (t, n) => new
                                                      {
                                                         TargetRating = t.Rating,
                                                         NeighborRating = n.Rating
                                                      })
                                       .ToList();
 
            // calculating numerator

            var sumOfProduct = combined.Sum(pair => pair.TargetRating\*pair.NeighborRating);

            var productOfSums = combined.Sum(pair => pair.TargetRating)\*

                                combined.Sum(pair => pair.NeighborRating);

            var numerator = sumOfProduct - (productOfSums / combined.Count);

            // calculating denominator

            var squareOfSum1 = Math.Pow(combined.Sum(pair => pair.TargetRating), 2);

            var sumOfSquare1 = combined.Sum(pair => pair.TargetRating\*pair.TargetRating);

            var sqaureRoot1 = Math.Sqrt(sumOfSquare1 - squareOfSum1/combined.Count);

            var squareOfSum2 = Math.Pow(combined.Sum(pair => pair.NeighborRating), 2);

            var sumOfSquare2 = combined.Sum(pair => pair.NeighborRating \* pair.NeighborRating);

            var sqaureRoot2 = Math.Sqrt(sumOfSquare2 - squareOfSum2 / combined.Count);

            var denominator = sqaureRoot1\*sqaureRoot2;

            return numerator/denominator;

        }
```

4. K-nearestneighbor

Этот алгоритм позволяет определять рекомендации на основании k ближайших соседей. Сначала считаем соседей и сортируем их по близости (можно использовать для подсчета любой из алгоритмов выше). Затем берем k первых соседей и рассчитываем проценты влияния каждого из соседей. Напр. k=3, коэффициенты влияния каждого **0.8 + 0.7 + 0.5 = 2.0** , тогда проценты 40%+35%+25% (отсчет идет от 2, т.е. 0.8/2 = 40%). Затем берем атрибуты каждого из пользователей (в данном случае все отмеченные ими песни) и высчитываем рейтинг каждой из них с использованием процента влияния:

Projected rating = (4.5 x 0.25) + (5 x 0.35) + (3.5 x 0.4)  = 4.27.

После этого сортируем и показываем как рекомендацию n посчитанных rating

**Использование:**

- если вх. данные сплошные (нет или мало пустых ячеек без цифр), то Minkowski

- если вх. данные имеют много пустых ячеек, то лучше Cosine Similarity

- если данные подвержены инфляции (grade-inflation), то лучше Pearson Correlation Coefficient. Инфляция это года Sam ставит оценки от 1 до 5, а Ben только 4 и 5, то есть 4 Бена может быть равна 2 или 3 Сэма. Чтобы это учесть лучше использовать PearsonCorrelation.

**ВЫВОД**

Основным недостатком всех этих алгоритмов является то, что они все не масштабируемы. Если есть 1000000 пользователей и 1000000 песен, то необходимо будет обработать всех пользователей, чтобы найти ближайшего соседа. Держать всех в памяти не реально и вычислять долго. Кроме того, будет много пробелов, т.к. каждый пользователь не мог оценить миллион песен. К тому же пользователи ленивы, не все ставят оценки, либо не обновляют уже проставленные, если речь, например, идет об оценках купленных товаров и пользователь разочаровался в товаре, но он уже не ухудшит оценку. А также пользователи могут лгать в своих оценках.
