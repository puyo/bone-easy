<script>
  let fetching = getDict()
  let pickupLines = []
  let count = 10
  let dict = new Map()
  const expand_re = /\{(\w+)\}/

  async function getDict() {
    const response = await fetch("bone-easy.txt")
    const data = await response.text()
    const lines = data.split("\n").forEach(line => {
      const [key, value] = line.split(/\s*-->\s*/)
      if (key && value) {
        const existing = dict.get(key) || []
        dict.set(key, [...existing, value])
      }
    })

    generate(dict, count)
  }

  function generate(dict, count) {
    for (let i = 0; i < count; ++i) {
      const line = expandRepeatedly(dict, "{SENTENCE}")
      pickupLines = [...pickupLines, line]
    }
  }


  function expandRepeatedly(dict, input) {
    let result = input
    // avoid infinite loop buy expanding at most 10,000 times
    for (let i = 0; i < 10000; i++) {
      const newResult = expandOnce(dict, result)
      if (newResult === result) {
        break
      } else {
        result = newResult
      }
    }
    return result.replace(/^\w/, (c) => c.toUpperCase())
  }

  function expandOnce(dict, input) {
    const output = input.replace(expand_re, (match, key) => {
      const options = dict.get(key)
      if (options) {
        return randomElem(options)
      } else {
        return `[${key}]`
      }
    })
    return output
  }

  function randomElem(array) {
    return array[Math.floor(Math.random() * array.length)]
  }

  function more() {
    generate(dict, count)
  }
</script>

<main>
  <h1>Bone-Easy<sup>(TM)</sup></h1>
  {#await fetching}
    <p>...loading...</p>
  {:then number}
    <ul class="pickup-lines">
      {#each pickupLines as line}
        <li>
          {line}
        </li>
      {/each}
    </ul>
    <button on:click={more}>{count} more</button>
  {:catch error}
    <p style="color: red">{error.message}</p>
  {/await}
</main>

<style>
  main {
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
