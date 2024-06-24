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

      exchangeRate = data.data[currencyTo];
      
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


<main>
  <h1>Конвертер валют на {new Date().toLocaleDateString()}  </h1>
  <label for='currencyFrom'>У меня есть
    <select bind:value={currencyFrom} id='currencyFrom' on:change={updateExchangeRate}>
  {#each currencies as currency}
  <option value={currency.code}>{currency.name} - {currency.code}</option>
  {/each}
</select>
</label>

<label>Сумма для конвертации
  <input type="text" bind:value={amountFrom} placeholder="Сумма для конвертации" on:input={updateExchangeRate} >
</label>

<label for='currencyTo'>Хочу купить
  <select bind:value={currencyTo} id='currencyTo' on:change={updateExchangeRate}>
  {#each currencies as currency}
  <option value={currency.code}>{currency.name} - {currency.code}</option>
  {/each}
</select>
</label>

<p id='amount'>
  Итого в {currencyTo} : 
  {#if isNaN(amountTo) }
    <div> Введите валюту, в которую вы хотите сконвертировать. </div>
  {:else if amountFrom === 0 }
    <div> Введите сумму, которую вы хотите сконвертировать. </div>
  {:else}
    <span>{amountTo} {currencyTo}</span>
  {/if}
</p>

</main>



<style>
  main {
    margin: 0 auto;
    padding: 10px 50px ;
    border-radius: 5px;
    background-color: white;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    gap : 10px;
    width: 450px;
    box-shadow: 13px 13px 46px 11px rgba(204,204,204,1);

  }
   h1{
    text-align: left;
    width: 200px;
    font-size: 20px;
    color:lightseagreen;
   }
  input {
    border-radius: 5px;
    color: black;
    border: 1px solid  lightseagreen;
    background-color: white;
    box-shadow: 5px 5px 5px lightseagreen;
    height: 20px;
    font-size: 10px; 

  }
  select {
    border-radius: 5px;
    color: black;
    border: 1px solid  lightseagreen;
    background-color: white;
    box-shadow: 5px 5px 5px lightseagreen;
    height: 20px;
    font-size: 12px; 

  }

  p {
    margin: 20px auto;
    width: 450px;
    font-size: 15px; 
    font-weight: 900;
    color: rgb(33, 178, 178);
    text-align: start;  
  }
  
  label {
      align-self: flex-start;      
      font-size: 15px;
      color: gray;
    }
    
  div{
      color: red;
      font-size: 10px;
      font-weight: 300;
     }

  span {
    color: gray;
     }

</style> 