<script lang='ts'>
import Freecurrencyapi from '@everapi/freecurrencyapi-js';
import { onMount } from 'svelte';
import {currencies}from './lib/currencies.svelte'

  let currencyFrom:string = '';
  let currencyTo:string = '';
  let amountFrom:number = 0;
  let amountTo:number= 0;
  let exchangeRate:number = 0;
  
  const freecurrencyapi = new Freecurrencyapi('fca_live_9lFFlAQDT9tj88KOOYp5FiQUo03CpoUeqkQTcXVT');

  async function fetchExchangeRate() {
    try {
      const response = await freecurrencyapi.latest({
        base_currency: currencyFrom,
        currencies: currencyTo,
        apikey:'fca_live_9lFFlAQDT9tj88KOOYp5FiQUo03CpoUeqkQTcXVT'
      });
 
      const data = await response;

      exchangeRate = (data.data[currencyTo]).toFixed(2);

    } catch (error) {
      console.error('Ошибка при выполнении запроса:', error);
    }
  }

  async function updateExchangeRate() {
    if (amountFrom > 0) {
    await fetchExchangeRate();
    amountTo = +(amountFrom * exchangeRate).toFixed(2);
  } else {
    amountTo = 0;
  }
     
  }

  onMount(async () => {
    await fetchExchangeRate();
  });

</script>


<main class="converter">
  <div class="converter__title-wrapper">
    <h1 class="converter__title">КОНВЕРТЕР ВАЛЮТ </h1>
    <p class="converter__paragraph">{new Date().toLocaleDateString()}</p>
  </div>

  <div class="converter__input-wrapper">
    <label for='sumInput' class="converter__label">Сумма для конвертации
    <input type="number" bind:value={amountFrom} placeholder="Сумма для конвертации" on:input={updateExchangeRate} id='sumInput' class="converter__input">
    </label>


    <label for='currencyFrom' class="converter__label">
    У меня есть
      <select bind:value={currencyFrom} id='currencyFrom' on:change={updateExchangeRate} class="converter__select">
      {#each currencies as currency}
         <option class="converter__option"value={currency.code}>{currency.name} - {currency.code} - {currency.flag}</option>
      {/each}
    </select>
  </label>

  <label for='currencyTo' class="converter__label">Хочу купить
    <select bind:value={currencyTo} id='currencyTo' on:change={updateExchangeRate} class="converter__select">
      {#each currencies as currency}
        <option value={currency.code}>{currency.name} - {currency.code} - {currency.flag}</option>
      {/each}
    </select>
  </label>

  {#if exchangeRate !== null && exchangeRate !== undefined && currencyFrom }
    <p class="converter__rate">1 {currencyFrom} = {exchangeRate} {currencyTo}</p>
  {:else}
    <p class="converter__rate"></p>
  {/if}


{#if amountFrom !== null && amountFrom !== 0 && currencyFrom}
  <p id='amount' class="converter__result">
    ИТОГО {amountFrom} {currencyFrom} : 

    {#if isNaN(amountTo) || !amountTo}
        <p class="converter__message">Введите валюту, в которую вы хотите сконвертировать.</p>
    {:else}
        <span class="converter__amount">{amountTo} {currencyTo}</span>
        <p class="converter__message"></p>

    {/if}
  </p>
{:else}
  <p id='amount' class="converter__result">
    ИТОГО : 
  {#if amountFrom === 0 || isNaN(amountFrom) || amountFrom === null }
     <p class="converter__message">Введите сумму, которую вы хотите сконвертировать.</p>
  {:else if !currencyFrom}
     <p class="converter__message">Введите валюту, из которой вы хотите сконвертировать.</p>
    {:else}
    <p class="converter__message"></p>
  {/if}

{/if}

</div>
</main>



<style>
  .converter {
    margin: 0 auto;
    padding: 40px 50px 30px 50px ;
    border-radius: 12px;
    background-color: var(--white-color);
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    gap : 10px;
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
    color:var(--main-color);
   }
   .converter__paragraph {
    display: flex;
    align-self: center;
    justify-content: center;
    padding: 5px;
    font-size: 14px;
    font-weight: 700;
    color:var(--white-color);
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
    border: 1px solid  var(--shadow-color);
    background-color: var(--white-color);
    box-shadow: 5px 5px 5px var(--shadow-color);
    height: 30px;
    font-size: 10px;
    padding-left: 10px;
    margin-top: 2px;
  }
  
  input::placeholder{
    font-size: 14px;
  }
  .converter__input:focus, .converter__input:active, .converter__input:visited{
    border: 1px solid var(--secondary-color);
  }
  .converter__select {
    border-radius: 5px;
    color: #272727;
    border: 1px solid  var(--shadow-color);
    background-color: var(--white-color);
    box-shadow: 5px 5px 5px var(--shadow-color);
    height: 30px;
    font-size: 12px;
    padding: 5px; 
    margin-top: 2px;
  }
  .converter__select:focus, .converter__select:active, .converter__select:visited{
    border: 1px solid var(--secondary-color);
  }  
  
  .converter__rate {
    height:22.5px;
    font-size: 16px;
    font-weight: 200;
    color: var(--main-color);
  }
  .converter__result {
    color: var(--main-color);
    font-size: 18px; 
    font-weight: 900;
  }
  .converter__rate, .converter__result {
    text-align: start; 
    margin:0;
    padding:0;
  }
  .converter__message {
      color: var(--error-color);
      font-size: 10px;
      font-weight: 300;
      height: 22.5px;
      margin: 10px  0 0 0 ;
    }

  .converter__amount {
       color: var(--secondary-color); 
       font-size: 18px;
       font-weight: 800;}

@media screen and (max-width: 440px) {
  .converter {
    width: 280px;
    box-sizing: border-box;
  }
 .converter__paragraph{
  padding: 5px;
    font-size: 10px;
    font-weight: 700; }
    0
 .converter__input-wrapper{
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