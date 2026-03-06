# 디자인 패턴 - 상태 패턴
![image](http://wiki.iad.zhdk.ch/wiki/download/attachments/2783379457/Screen%20Shot%202022-10-06%20at%2013.30.05.png?version=1&modificationDate=1665055903007&cacheVersion=1&api=v2)
- 상태 패턴은 Finite State Machine 을 베이스로 생각하면 쉽다.
- 결국 interface 로 각 상태들을 추상화하고, 각 상태를 구현체로 다시 나타내는 것.
	- 외부에서는 어떤 상태인지 알 필요 없이 상태에게 상태를 맡김.
- 같은 메서드가 상태마다 다르게 동작해야 할 때
- 상태 전환간 조건이 붙을 때 (아무 상태에서나 전환 불가)

## 상태 패턴 도입 이전 코드
```kotlin
class Player {  
    private val cards: MutableList<Card> = mutableListOf()  
    private var gameState: GameState = GameState.INIT  
  
    fun receiveCard(card: Card) {  
        when (gameState) {  
            GameState.STAY, GameState.BUST, GameState.BLACKJACK ->  
                throw IllegalStateException("턴이 종료되어 카드를 받을 수 없습니다.")  
            GameState.INIT -> {  
                cards.add(card)  
                if (cards.size == 2) {  
                    gameState = if (score() == 21) GameState.BLACKJACK else GameState.HIT  
                }  
            }  
            GameState.HIT -> {  
                cards.add(card)  
                if (score() > 21) gameState = GameState.BUST  
            }  
        }  
    }  
  
    fun stay() {  
        when (gameState) {  
            GameState.HIT -> gameState = GameState.STAY  
            GameState.INIT -> throw IllegalStateException("초기 상태에서는 stay 할 수 없습니다.")  
            GameState.STAY, GameState.BUST, GameState.BLACKJACK ->  
                throw IllegalStateException("턴이 종료되어 stay 상태로 변경할 수 없습니다.")  
        }  
    }  
  
    fun isFinished(): Boolean = when (gameState) {  
        GameState.STAY, GameState.BUST, GameState.BLACKJACK -> true  
        GameState.INIT, GameState.HIT -> false  
    }  
  
    fun cards(): List<Card> = cards.toList()  
  
    fun score(): Int {  
        val baseScore = cards.sumOf { it.number.score }  
        val countOfAce = cards.count { it.number == CardNumber.ACE }  
        var score = baseScore  
        repeat(countOfAce) {  
            if (score + 10 <= 21) score += 10  
        }  
        return score  
    }  
  
    fun judgementGameResult(other: Player): GameResult {  
        return when (gameState) {  
            GameState.BLACKJACK -> {  
                if (other.gameState == GameState.BLACKJACK) GameResult.DRAW  
                else GameResult.BLACKJACK_WIN  
            }  
            GameState.BUST -> GameResult.LOSE  
            GameState.STAY -> when {  
                other.gameState == GameState.BLACKJACK -> GameResult.LOSE  
                other.gameState == GameState.BUST -> GameResult.WIN  
                score() > other.score() -> GameResult.WIN  
                score() < other.score() -> GameResult.LOSE  
                else -> GameResult.DRAW  
            }  
            GameState.INIT, GameState.HIT ->  
                throw IllegalStateException("게임이 진행중인 상태에서는 승패 비교를 할 수 없습니다.")  
        }  
    }  
}
```


## 상태패턴 도입 이후 코드
```kotlin
sealed class AfterInit(val cards: Cards) : State {  
  
    override fun cards(): Cards = cards  
  
    override fun score(): Score = cards.score()  
}

sealed class Finished(cards: Cards) : AfterInit(cards) {  
  
    override fun receiveCard(card: Card): State = throw IllegalStateException("턴이 종료되어 카드를 받을 수 없습니다.")  
  
    override fun stay(): State = throw IllegalStateException("턴이 종료되어 stay 상태로 변경할 수 없습니다.")  
  
    override fun isFinished(): Boolean = true  
}

class Blackjack(cards: Cards) : Finished(cards) {  
    override fun judgementGameResult(otherScore: Score): GameResult {  
        return if (otherScore.isBlackjack) {  
            GameResult.DRAW  
        } else {  
            GameResult.BLACKJACK_WIN  
        }  
    }  
}

class Bust(cards: Cards) : Finished(cards) {  
    override fun judgementGameResult(otherScore: Score): GameResult = GameResult.LOSE  
}

class Stay(cards: Cards) : Finished(cards) {  
    override fun judgementGameResult(otherScore: Score): GameResult {  
        return when {  
            otherScore.isBlackjack -> GameResult.LOSE  
            otherScore.isBust -> GameResult.WIN  
            else -> this.score().compareGameResult(other = otherScore)  
        }  
    }  
}

sealed class Running(cards: Cards) : AfterInit(cards) {  
  
    override fun isFinished(): Boolean = false  
  
    override fun judgementGameResult(otherScore: Score): GameResult {  
        throw IllegalStateException("게임이 진행중인 상태에서는 승패 비교를 할 수 없습니다.")  
    }  
}

class Hit(cards: Cards) : Running(cards) {  
    override fun receiveCard(card: Card): State {  
        val cards = this.cards.receiveCard(card = card)  
  
        return when (cards.isBustScore) {  
            true -> Bust(cards)  
            false -> Hit(cards)  
        }  
    }  
  
    override fun stay(): State = Stay(this.cards)  
}
```

- 새로운 상태가 추가된다?
	- 기존 클래스 파일이나 코드를 모두 열어볼 필요 없이, 새로운 클래스 파일만 만들어서 추가하면 끝!
	- 상태 이전만 체크해주면 된다.


- '전략 패턴이랑 비슷한데?'
	- 전략 패턴은 Runtime에 동적으로 코드를 관리하기 위해
	- 상태 패턴은 상태를 관리하는 구현체에게 완전히 일임(Encapsulation)하기 위해
