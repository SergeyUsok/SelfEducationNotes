# Data Mining

## The Ancient Art of the Numerati (Data Mining) [http://guidetodatamining.com/](http://guidetodatamining.com/)

1. 1) **Создаем систему рекомендаций, основанную на оценках пользователей**

**(****User ****-**** based****).**

Допустим, есть таблица с пользователями, песнями и их оценками (от 1 до 5):

|   | Unforgiven | Paradise | Ghost | Sniper |
| --- | --- | --- | --- | --- |
| Ben | 1.5 |   |   | 4 |
| Tom | 3 | 5 |   |   |
| Ron | 3.5 |   | 2 | 4.5 |
| Sam |   | 3 | 5 |   |

Алгоритмы:

1. Minkowski Distance Metric

 ![](data:image/*;base64,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)

- Euclidean Distance when r=2

- Manhattan Distance when r=1

publicstaticdouble MinkowskiDistance(User target, User neighbor, double r)

{

    var combinedBySongs = target.Songs.Join(neighbor.Songs,

    s => s.Name, s => s.Name,

                          (t, n) => new { TargetRating = t.Rating, NeighborRating = n.Rating });

    var sum = combinedBySongs.Sum(p => Math.Pow(Math.Abs(p.TargetRating - p.NeighborRating), r));

    returnMath.Pow(sum, 1/r);

}

4. Cosine Similarity

 ![](data:image/*;base64,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)

3. Pearson Correlation Coefficient

 ![](data:image/*;base64,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)

Результат от -1 до 1, где 1 прямая корреляция и схожесть

Реализованный алгоритм ниже:



        publicstaticdouble PearsonCorrelation(User target, User neighbor)

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

4. K-nearestneighbor

Этот алгоритм позволяет определять рекомендации на основании k ближайших соседей. Сначала считаем соседей и сортируем их по близости (можно использовать для подсчета любой из алгоритмов выше). Затем берем k первых соседей и рассчитываем проценты влияния каждого из соседей. Напр. k=3, коэффициенты влияния каждого **0.8 + 0.7 + 0.5 = 2.0** , тогда проценты 40%+35%+25% (отсчет идет от 2, т.е. 0.8/2 = 40%). Затем берем атрибуты каждого из пользователей (в данном случае все отмеченные ими песни) и высчитываем рейтинг каждой из них с использованием процента влияния:

Projected rating = (4.5 x 0.25) + (5 x 0.35) + (3.5 x 0.4)  = 4.27.

После этого сортируем и показываем как рекомендацию n посчитанных rating

**Использование:**

- если вх. данные сплошные (нет или мало пустых ячеек без цифр), то Minkowski

- если вх. данные имеют много пустых ячеек, то лучше Cosine Similarity

- еслиданныеподверженыинфляции (grade-inflation), толучше Pearson Correlation Coefficient. Инфляция это года Sam ставит оценки от 1 до 5, а Ben только 4 и 5, то есть 4 Бена может быть равна 2 или 3 Сэма. Чтобы это учесть лучше использовать PearsonCorrelation.

**ВЫВОД**

Основным недостатком всех этих алгоритмов является то, что они все не масштабируемы. Если есть 1000000 пользователей и 1000000 песен, то необходимо будет обработать всех пользователей, чтобы найти ближайшего соседа. Держать всех в памяти не реально и вычислять долго. Кроме того, будет много пробелов, т.к. каждый пользователь не мог оценить миллион песен. К тому же пользователи ленивы, не все ставят оценки, либо не обновляют уже проставленные, если речь, например, идет об оценках купленных товаров и пользователь разочаровался в товаре, но он уже не ухудшит оценку. А также пользователи могут лгать в своих оценках.
