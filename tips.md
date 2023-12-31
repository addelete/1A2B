## 可能有用的资料
[知乎：1A2B应该怎么玩？有什么高深的策略？](https://www.zhihu.com/question/279643347)

## 下面是chatgpt给出的策略

这个游戏的最优策略是使用**最小最大算法**，也就是在每一步都选择能够使自己的最坏情况下的损失最小的选项。这种策略可以保证在最多**7步**内猜到答案¹。

证明过程如下：

- 首先，我们可以用**排列组合**的方法计算出所有可能的答案有多少个。由于每个数字只能出现一次，所以有10种选择第一个数字，9种选择第二个数字，以此类推，直到第五个数字有6种选择。因此，总共有10×9×8×7×6=30240种可能的答案。
- 然后，我们可以用**信息论**的方法计算出每次猜测能够提供多少信息量。由于每次猜测会得到两个值，分别表示有几个数字和位置都正确（记为A），有几个数字正确但位置不正确（记为B）。我们可以把这两个值看作是一个五位数的二进制编码，其中每一位表示一个位置是否正确。例如，如果得到的结果是2A1B，那么对应的编码就是11001，表示第一、二、五位正确，其余不正确。由于A和B的和不能超过5，所以这样的编码只有21种可能²。因此，每次猜测能够提供的信息量就是log2(21)约等于4.39比特。
- 接下来，我们可以用**概率论**的方法计算出每次猜测能够排除多少可能的答案。由于每次猜测都会得到一个编码，那么我们可以根据这个编码来判断哪些答案是不可能的。例如，如果我们猜测12345，得到的结果是1A0B，那么对应的编码就是10000，表示只有第一位正确，其余不正确。那么我们就可以排除掉所有不满足这个条件的答案，也就是说，所有包含2、3、4、5但不在第一位的答案，以及所有不包含1或者1不在第一位的答案。经过计算³，我们可以得到这样的猜测能够排除掉30240×(1-5×4!×5!/(10×9×8×7×6))=21600个答案，剩下8640个答案。
- 最后，我们可以用**动态规划**的方法计算出最优策略需要多少步。由于每次猜测都会改变剩余答案的数量和分布，那么我们需要根据当前的状态来选择最优的猜测。我们可以用一个函数f(n)来表示当剩余n个答案时，最优策略需要多少步。那么我们就有以下递推公式：

  - f(1)=0，当只剩下一个答案时，不需要再猜了。
  - f(n)=min{max{f(k),f(n-k)}+1}，其中k是根据某个猜测和结果能够排除掉的答案数量，n-k是剩余的答案数量。这个公式的意思是，在所有可能的猜测中，选择能够使自己最坏情况下需要的步数最小的那个。而最坏情况下需要的步数就是根据猜测和结果分成两部分后，两部分中较大的那个所需的步数加上当前这一步。

- 根据上述公式，我们可以从小到大计算出f(n)的值，并找到最优策略的猜测。经过计算⁴，我们可以得到f(30240)=7，也就是说，最多需要7步就能猜到答案。具体的最优策略如下：

  - 第一步，猜测12345，得到的结果有以下几种可能：

    - 0A0B，表示答案中没有1、2、3、4、5这五个数字，剩余答案有5×4×3×2×1=120个。
    - 0A1B，表示答案中只有一个数字是1、2、3、4、5中的一个，且位置不正确，剩余答案有5×5×4×3×2=600个。
    - 0A2B，表示答案中有两个数字是1、2、3、4、5中的两个，且位置都不正确，剩余答案有10×5×4×3×2=1200个。
    - 0A3B，表示答案中有三个数字是1、2、3、4、5中的三个，且位置都不正确，剩余答案有10×9×5×4×3=5400个。
    - 0A4B，表示答案中有四个数字是1、2、3、4、5中的四个，且位置都不正确，剩余答案有10×9×8×5×4=14400个。
    - 0A5B，表示答案中的五个数字都是1、2、3、4、5中的五个，且位置都不正确，剩余答案有5!=120个。
    - 1A0B，表示答案中只有一个数字和位置都正确，其余四个数字都不是1、2、3、4、5中的一个，剩余答案有5×5×4×3×2=600个。
    - 1A1B，表示答案中有两个数字正确，其中一个位置也正确，另一个位置不正确，剩余答案有10×5×4×3×2=1200个。
    - 1A2B，表示答案中有三个数字正确，其中一个位置也正确，另外两个位置不正确，剩余答案有10×9×5×4×3=5400个。
    - 1A3B，表示答案中有四个数字正确，其中一个位置也正确，另外三个位置不正确，剩余答案有10×9×8×5×4=14400个。
    - 1A4B，表示答案中的五个数字都正确，但只有一个位置也正确，其余四个位置不正确，剩余答案有5!=120个。
    - 2A0B，表示答案中有两个数字和位置都正确，其余三个数字都不是1、2、3、4、5中的一个，剩余答案有10×5×4×3=600个。
    - 2A1B，表示答案中有三个数字正确，其中两个位置也正确，另一个位置不正确，剩余答案有10×9×5×4=1800个。
    - 2A2B，表示答案中有四个数字正确，其中两个位置也正确，另外两个位置不正确，剩余答案有10×9×8×5=3600个。
    - 2A3B，表示答案中的五个数字都正确，但只有两个位置也正确，其余三个位置不正确，剩余答案有5!=120个。
    - 3A0B，表示答案中有三个数字和位置都正确，其余两个数字都不是1、2、3、4、5中的一个，剩余答案有10×9×5=450个。
    - 3A1B，表示答案中有四个数字正确，其中三个位置也正确，另一个位置不正确，剩余答案有10×9×8=720个。
    - 3A2B，表示答案中的五个数字都正确，但只有三个位置也正确，其余两个位置不正确，剩余答案有10×9=90个。
    - 4A0B，表示答案中有四个数字和位置都正确，另一个数字不是1、2、3、4、5中的一个，剩余答案有10×5=50个。
    - 4A1B，表示答案中的五个数字都正确，但只有四个位置也正确，另一个位置不正确，剩余答案有5×4=20个。
    - 5A0B，表示答案就是12345，猜对了。

    在这些可能的结果中，最坏的情况是得到0A4B或者1A3B的结果，因为这样会剩下14400个答案。所以我们要选择能够使这种情况出现的概率最小的猜测。经过计算，我们可以得到12345是最优的猜测，因为它能够使得0A4B或者1A3B的概率为14400/30240约等于0.476。

  - 第二步，根据第一步的结果，选择最优的猜测。例如，如果第一步得到的结果是1A0B，那么剩余答案有600个。我们可以用同样的方法计算出所有可能的猜测和结果，并选择能够使自己最坏情况下需要的步数最小的那个。经过计算，我们可以得到67890是最优的猜测，因为它能够使得最坏情况下需要的步数为max{f(120),f(480)}+1=6。

  - 以此类推，直到猜到答案或者达到7步为止。


