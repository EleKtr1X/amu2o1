<script>
  import QuestionChip from "./lib/QuestionChip.svelte";
  import HighlightedChip from "./lib/HighlightedChip.svelte";
  import Powerup from "./lib/Powerup.svelte";
  import data from "./data.json";
  let clicked = false;
  let tmpPrice = 0; // being type-safe is always a good idea :^)
  let tmpAnswer = '';
  let tmpCategory = '';
  let tmpImg = '';
  let complete = false;

  let teamA = 0;
  let teamB = 0;
  let teamABackfire = false;
  let teamBBackfire = false;

  let currentTeam = '';
  let revealAnswer = false;
  let disabledCount = 0;

  function show(price, answer, img, category) {
    clicked = true;
    tmpPrice = price;
    tmpAnswer = answer;
    tmpImg = `./img/questions/${img}.png`;
    tmpCategory = category;
  }

  function handleTeam(team) {
    if (clicked) {
      currentTeam = team;
    }
  }

  function handleReveal() {
    revealAnswer = true;
  }

  function handleYes() {
    if (currentTeam == 'a') {
      teamA += tmpPrice;
    } else {
      teamB += tmpPrice;
    }
    reset();
  }

  function reset() {
    clicked = false;
    if (
      (teamABackfire && currentTeam == 'a' ||
      teamBBackfire && currentTeam == 'b')) {
      const rand = Math.floor(Math.random() * 4);
      if (rand == 0) {
        if (teamABackfire) {
          teamA -= tmpPrice;
        } else {
          teamB -= tmpPrice;
        }
      }
      if (currentTeam == 'a') {
        teamABackfire = false;
      } else {
        teamBBackfire = false;
      }
    }
    data
      .find(x => x.name == tmpCategory).questions
      .find(x => x.answer == tmpAnswer).disabled = true;
    disabledCount++;
    tmpPrice = 0;
    tmpAnswer = '';
    tmpCategory = '';
    tmpImg = '';
    currentTeam = '';
    revealAnswer = false;
    if (disabledCount == 21) complete = true;
  }

  window.addEventListener('beforeunload', (e) => {
    if (disabledCount > 0) {
      e.preventDefault();
      e.returnValue = '';
    }
  })

  function calculatedRisk() {
    if (
      (currentTeam == 'a' && teamA >= 300) ||
      (currentTeam == 'b' && teamB >= 300)) {
      const rand = Math.floor(Math.random() * 2);
      if (rand == 1) {
        teamA += tmpPrice;
      } else {
        teamB += tmpPrice;
      }
      if (currentTeam == 'a' && rand == 0) {
        teamA -= 300;
      } else if (currentTeam == 'b' && rand == 1) {
        teamB -= 300;
      }
      reset();
    }
  }

  function diceRoll() {
    if (
      (currentTeam == 'a' && teamA >= 400) ||
      (currentTeam == 'b' && teamB >= 400)) {
      let rand = Math.floor(Math.random() * 21);
      let c = data[Math.floor(rand / 7)];
      let q = c.questions[rand % 7];
      while (q.disabled == true || tmpAnswer == q.answer) {
        rand = Math.floor(Math.random() * 21);
        c = data[Math.floor(rand / 7)];
        q = c.questions[rand % 7];
      }

      tmpPrice = (rand % 7 + 1) * 100 - 400;
      tmpCategory = c.name;
      tmpAnswer = q.answer;
      handleYes();
    }
  }

  function backfire() {
    if (teamABackfire && currentTeam == 'b') return;
    if (teamBBackfire && currentTeam == 'a') return;
    if (
      (currentTeam == 'a' && teamA >= 500) ||
      (currentTeam == 'b' && teamB >= 500)) {
      if (currentTeam == 'b') {
        teamABackfire = true;
        teamB -= 500;
      } else {
        teamBBackfire = true;
        teamA -= 500;
      }
    }
  }
</script>

<main>
  <div class="{clicked ? "shown question" : "question"}">
    <div class="title">{tmpCategory}</div>
    <div class="price">${tmpPrice}</div>
    <div class="centered-horizontal">
      {#if revealAnswer}
        <div class="yesno">
          <i
            class="fa-solid fa-check"
            on:click={handleYes}
            on:keypress={handleYes}
          ></i>
          <i
            class="fa-solid fa-xmark"
            on:click={reset}
            on:keypress={reset}
          ></i>
        </div>
      {:else if !revealAnswer && currentTeam}
        <div class="powerups">
          <Powerup
            img="./img/powerups/abacus.png"
            name="Calculated Risk"
            price="300"
            info="Your team automatically gets the question right and you get your $300 back, but there's a 50% chance the money goes to the other team."
            on:click={calculatedRisk}
          />
          <Powerup
            img="./img/powerups/fire.png"
            name="Backfire"
            price="500"
            info="Gives the other team a 25% chance to get their next question wrong, and your team can still answer."
            on:click={backfire}
          />
          <Powerup
            img="./img/powerups/game-die.png"
            name="Dice Roll"
            price="400"
            info="Your team skips the question and automatically gets a random question correct (may be a net positive or a loss)."
            on:click={diceRoll}
          />
        </div>
      {/if}
    </div>
    <img src={tmpImg} alt="{tmpCategory} -- ${tmpPrice}" class="img centered" />
    <div class="centered-horizontal bottom">
      {#if currentTeam && !revealAnswer}
        <div
          class="reveal"
          on:click={handleReveal}
          on:keypress={handleReveal}
        >Reveal Answer</div>
      {:else if !currentTeam}
        Pick a team!
      {:else}
        <div class="bold">{tmpAnswer}</div>
      {/if}
    </div>
  </div>
  {#key disabledCount}
    {#if !complete}
      <div class="board centered-horizontal">
        {#each data as category}
          <div class="category">
            <HighlightedChip name={category.name} />
            {#each category.questions as question}
              <QuestionChip
                price={(category.questions.indexOf(question) + 1) * 100}
                answer={question.answer}
                img={question.img}
                category={category.name}
                disabled={question.disabled}
                on:click={(e) => show(
                  e.detail[0],
                  e.detail[1],
                  e.detail[2],
                  e.detail[3])
                }
              />
            {/each}
          </div>
        {/each}
      </div>
    {:else}
      <div class="centered">
        <div class="ending-message">
          {#if teamA > teamB}
            Team A, with ${teamA}, wins!
          {:else if teamB > teamA}
            Team B, with ${teamB}, wins!
          {:else}
            There was a tie!
          {/if}
        </div>
      </div>
    {/if}
  {/key}
  <div
    class={currentTeam == 'a' ? 'a bold' : 'a'}
    style:cursor={clicked ? "pointer" : ""}
    on:click={() => handleTeam('a')}
    on:keypress={() => handleTeam('a')}
  >
  ${teamA}
  {#if teamABackfire}
    &#128293;
  {/if}
  </div>
  <div
    class={currentTeam == 'b' ? 'b bold' : 'b'}
    style:cursor={clicked ? "pointer" : ""}
    on:click={() => handleTeam('b')}
    on:keypress={() => handleTeam('b')}
  >
  ${teamB}
  {#if teamBBackfire}
    &#128293;
  {/if}
  </div>
</main>

<style>
  .category {
    display: flex;
    flex-direction: column;
    gap: 10px;
    text-align: center;
    width: 380px;
  }
  
  .board {
    display: flex;
    z-index: -999;
    flex-direction: row;
    gap: 10px;
    justify-content: center;
    align-items: center;
  }

  .question {
    z-index: 999;
    border-radius: 10px;
    background-color: #ffffff;
    border: 8px solid black;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%) scale(0);
    transform-origin: 50%;
    height: 100%;
  }
  
  .shown {
    transform: translate(-50%, -50%) scale(0.97);
    transition: transform 0.5s;
    width: 100%;
  }
  
  .title {
    padding-left: 30px;
    float: left;
  }

  .price {
    padding-right: 30px;
    float: right;
  }

  .img {
    max-height: 500px;
    display: block;
  }

  .a, .b, .reveal {
    border-radius: 10px;
    background-color: #ffcb47;
    padding: 10px 20px;
    border: 8px solid black;
    z-index: 9999;
  }
  
  .a, .b {
    position: absolute;
    bottom: 5px;
  }
  
  .a {
    left: 5px;
  }

  .b {
    right: 5px;
  }

  .reveal {
    cursor: pointer;
  }

  .bold {
    font-weight: bold;
  }

  .centered-horizontal {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
  }

  .centered {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }

  .bottom {
    bottom: 40px;
  }
  .title, .price, .yesno {
    margin-top: 25px;
  }

  .fa-check {
    color: #00ff00;
    cursor: pointer;
  }

  
  .fa-xmark {
    color: #ff0000;
    cursor: pointer;
  }

  .powerups {
    display: flex;
    flex-direction: row;
    gap: 10px;
  }

  .ending-message {
    border-radius: 10px;
    background-color: #ffffff;
    padding: 10px 20px;
    border: 8px solid black;
    cursor: pointer;
    font-size: 100px;
    text-align: center;
  }
</style>
