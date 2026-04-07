<script lang="ts">
  import Freecurrencyapi from '@everapi/freecurrencyapi-js';
  import { onMount } from 'svelte';
  import { currencies } from './lib/currencies';
  import { API_KEY } from './lib/constants';
  interface CurrencyData {
    [key: string]: number;
  }

  interface ApiResponse {
    data: CurrencyData;
  }

  interface CacheEntry {
    rate: number;
    timestamp: number;
  }

  const CACHE_TTL_MS = 5 * 60 * 1000; // 5 минут
  const rateCache = new Map<string, CacheEntry>();

  const freecurrencyapi = new Freecurrencyapi(API_KEY);

  let currencyFrom: string = 'USD';
  let currencyTo: string = 'EUR';
  let amountFrom: number = 0;
  let amountTo: number = 0;
  let exchangeRate: number = 0;
  let isLoading: boolean = false;
  let errorMessage: string = '';

  let debounceTimer: ReturnType<typeof setTimeout> | undefined;

  async function fetchExchangeRateWithCache(): Promise<number> {
    if (!currencyFrom || !currencyTo) return 0;
    if (currencyFrom === currencyTo) return 1;

    const cacheKey = `${currencyFrom}_${currencyTo}`;
    const now = Date.now();
    const cached = rateCache.get(cacheKey);
    if (cached && now - cached.timestamp < CACHE_TTL_MS) {
      console.debug(`[Cache] using cached rate for ${cacheKey}: ${cached.rate}`);
      return cached.rate;
    }

    console.debug(`[Cache] fetching rate for ${cacheKey}`);
    isLoading = true;
    errorMessage = '';

    try {
      const response = (await freecurrencyapi.latest({
        base_currency: currencyFrom,
        currencies: currencyTo,
      })) as ApiResponse;

      const rate = response.data[currencyTo];
      rateCache.set(cacheKey, { rate, timestamp: now });
      console.log(rateCache);
      return rate;
    } catch (error) {
      console.error('Ошибка при запросе курса:', error);
      errorMessage = 'Не удалось получить курс. Проверьте соединение.';
      return 0;
    } finally {
      isLoading = false;
    }
  }

  async function recalc() {
    if (!currencyFrom || !currencyTo) return;

    if (currencyFrom === currencyTo) {
      exchangeRate = 1;
      updateAmountTo();
      return;
    }

    const rate = await fetchExchangeRateWithCache();
    if (rate > 0) {
      exchangeRate = rate;
      updateAmountTo();
    } else if (errorMessage) {
      exchangeRate = 0;
      amountTo = 0;
    }
  }

  function updateAmountTo() {
    if (currencyFrom === currencyTo) {
      amountTo = amountFrom;
      return;
    }

    if (amountFrom > 0 && exchangeRate > 0) {
      amountTo = Number((amountFrom * exchangeRate).toFixed(2));
    } else {
      amountTo = 0;
    }
  }

  function onAmountInput() {
    if (debounceTimer) clearTimeout(debounceTimer);
    debounceTimer = setTimeout(() => {
      updateAmountTo();
    }, 300);
  }

  $: if (currencyFrom && currencyTo) {
    recalc();
  }

  $: if (amountFrom !== undefined) {
    onAmountInput();
  }

  onMount(() => {
    recalc();
  });
</script>

<main class="converter">
  <div class="converter__title-wrapper">
    <h1 class="converter__title">КОНВЕРТЕР ВАЛЮТ</h1>
    <p class="converter__paragraph">{new Date().toLocaleDateString()}</p>
  </div>

  <div class="converter__input-wrapper">
    <label for="sumInput" class="converter__label">
      Сумма для конвертации
      <input
        type="number"
        bind:value={amountFrom}
        placeholder="Введите сумму"
        id="sumInput"
        class="converter__input"
        disabled={isLoading}
      />
    </label>

    <label for="currencyFrom" class="converter__label">
      У меня есть
      <select
        bind:value={currencyFrom}
        id="currencyFrom"
        class="converter__select"
        disabled={isLoading}
      >
        {#each currencies as currency}
          <option value={currency.code}>
            {currency.name} - {currency.code} - {currency.flag}
          </option>
        {/each}
      </select>
    </label>

    <label for="currencyTo" class="converter__label">
      Хочу купить
      <select
        bind:value={currencyTo}
        id="currencyTo"
        class="converter__select"
        disabled={isLoading}
      >
        {#each currencies as currency}
          <option value={currency.code}>
            {currency.name} - {currency.code} - {currency.flag}
          </option>
        {/each}
      </select>
    </label>

    {#if !isLoading && exchangeRate > 0 && currencyFrom && currencyTo}
      <p class="converter__rate">
        1 {currencyFrom} = {exchangeRate.toFixed(4)}
        {currencyTo}
      </p>
    {:else if isLoading}
      <p class="converter__rate converter__rate--loading">Загрузка курса...</p>
    {:else if errorMessage}
      <p class="converter__rate converter__rate--error">{errorMessage}</p>
    {:else}
      <p class="converter__rate"></p>
    {/if}

    <div class="converter__result-wrapper">
      <div class="converter__result-label">ИТОГО</div>
      {#if amountFrom > 0 && currencyFrom && currencyTo}
        <div class="converter__result">
          {amountFrom}
          {currencyFrom} =
          {#if !isNaN(amountTo) && amountTo > 0}
            <span class="converter__amount">{amountTo} {currencyTo}</span>
          {:else if isLoading}
            <span class="converter__message">⏳ Конвертируем...</span>
          {:else}
            <span class="converter__message">Невозможно рассчитать</span>
          {/if}
        </div>
      {:else}
        <div class="converter__result">
          <span class="converter__message">
            {#if amountFrom === 0 || isNaN(amountFrom)}
              Введите сумму для конвертации
            {:else if !currencyFrom}
              Выберите валюту "У меня есть"
            {:else if !currencyTo}
              Выберите валюту "Хочу купить"
            {/if}
          </span>
        </div>
      {/if}
    </div>
  </div>
</main>

<style>
  .converter {
    margin: 0 auto;
    padding: 40px 50px 30px 50px;
    border-radius: 12px;
    background-color: var(--white-color);
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    gap: 10px;
    width: 300px;
    box-shadow: var(--background-shadow);
  }

  .converter__title-wrapper {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 30px;
  }
  .converter__title {
    text-align: left;
    font-weight: 900;
    font-size: 18px;
    color: var(--main-color);
  }
  .converter__paragraph {
    display: flex;
    align-self: center;
    justify-content: center;
    padding: 5px;
    font-size: 14px;
    font-weight: 700;
    color: var(--white-color);
    background-color: var(--secondary-color);
    border-radius: 8px;
    text-align: center;
  }

  .converter__input-wrapper {
    display: flex;
    flex-direction: column;
    gap: 20px;
  }

  .converter__label {
    display: flex;
    flex-direction: column;
    align-self: flex-start;
    font-size: 14px;
    color: var(--gray-color);
    max-width: 228px;
    width: 100%;
    font-weight: 200;
  }
  .converter__input {
    box-sizing: border-box;
    border-radius: 5px;
    color: var(--black-color);
    border: 1px solid var(--shadow-color);
    background-color: var(--white-color);
    box-shadow: 5px 5px 5px var(--shadow-color);
    height: 30px;
    font-size: 10px;
    padding-left: 10px;
    margin-top: 2px;
  }

  input::placeholder {
    font-size: 14px;
  }
  .converter__input:focus,
  .converter__input:active,
  .converter__input:visited {
    border: 1px solid var(--secondary-color);
  }
  .converter__select {
    border-radius: 5px;
    color: #272727;
    border: 1px solid var(--shadow-color);
    background-color: var(--white-color);
    box-shadow: 5px 5px 5px var(--shadow-color);
    height: 30px;
    font-size: 12px;
    padding: 5px;
    margin-top: 2px;
  }
  .converter__select:focus,
  .converter__select:active,
  .converter__select:visited {
    border: 1px solid var(--secondary-color);
  }

  .converter__rate {
    height: 22.5px;
    font-size: 16px;
    font-weight: 200;
    color: var(--main-color);
  }
  .converter__result {
    color: var(--main-color);
    font-size: 18px;
    font-weight: 900;
  }
  .converter__rate,
  .converter__result {
    text-align: start;
    margin: 0;
    padding: 0;
  }
  .converter__message {
    color: var(--error-color);
    font-size: 10px;
    font-weight: 300;
    height: 22.5px;
    margin: 10px 0 0 0;
  }

  .converter__amount {
    color: var(--secondary-color);
    font-size: 18px;
    font-weight: 800;
  }

  @media screen and (max-width: 440px) {
    .converter {
      width: 280px;
      box-sizing: border-box;
    }
    .converter__paragraph {
      padding: 5px;
      font-size: 10px;
      font-weight: 700;
    }

    .converter__input-wrapper {
      gap: 15px;
    }

    .converter__amount {
      word-break: break-all;
      display: flex;
    }
    .converter__result {
      word-break: break-all;
    }
  }
</style>
