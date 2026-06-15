# Counter-va-Memo-factory-lar
// 1. Counter Factory — Max/Min va qadam sozlamalari bilan
const createCounter = ({ step = 1, min = -Infinity, max = Infinity } = {}) => {
  let count = 0; // Private state

  return {
    increment: () => {
      if (count + step <= max) {
        count += step;
      }
      return count;
    },
    decrement: () => {
      if (count - step >= min) {
        count -= step;
      }
      return count;
    },
    getValue: () => count,
    reset: () => {
      count = 0;
      return count;
    }
  };
};

// 2. Memoize — Funksiya natijalarini keshda saqlash (Xavfsiz Map bilan)
const memoize = (funk) => {
  const cache = new Map(); // Private cache state

  return (...args) => {
    const key = JSON.stringify(args); // Argumentlarni kalitga aylantiramiz
    if (cache.has(key)) {
      return cache.get(key); // Keshdan o'qish
    }
    const result = funk(...args);
    cache.set(key, result); // Keshga yozish
    return result;
  };
};

// 3. Once — Funksiyani faqat bir marta bajarishga majburlash
const once = (funk) => {
  let isCalled = false; // Private flag
  let result;           // Private kesh

  return (...args) => {
    if (!isCalled) {
      result = funk(...args);
      isCalled = true;
    }
    return result; // Keyingi chaqiriqlarda faqat birinchi natija qaytadi
  };
};

// 4. Debounce — Ketma-ket chaqiriqlarni kechiktirish
const debounce = (funk, ms) => {
  let timerId = null; // Private timer state

  return (...args) => {
    if (timerId) {
      clearTimeout(timerId); // Oldingi taymerni o'chirish
    }
    timerId = setTimeout(() => {
      funk(...args);
    }, ms);
  };
};
// Oddiy rekurssiv Fibonacci — O(2^n) murakkablikda
const plainFibonacci = (n) => (n <= 1 ? n : plainFibonacci(n - 1) + plainFibonacci(n - 2));

// Memoize qilingan Fibonacci — O(n) murakkablikda
const memoizedFibonacci = memoize((n) => 
  n <= 1 ? n : memoizedFibonacci(n - 1) + memoizedFibonacci(n - 2)
);

const N = 35; // Sinov qiymati

console.log(`=== TAQQOSLASH TESTI (N = ${N}) ===\n`);

// 1. Oddiy rekurssiya tezligi
console.time("1. Oddiy Rekurssiya Vaqti");
const res1 = plainFibonacci(N);
console.timeEnd("1. Oddiy Rekurssiya Vaqti");
console.log(`Natija: ${res1}\n`);

// 2. Kesh bilan ishlash tezligi
console.time("2. Memoize (Kesh) Bilan Ilk Chaqiriq");
const res2 = memoizedFibonacci(N);
console.timeEnd("2. Memoize (Kesh) Bilan Ilk Chaqiriq");
console.log(`Natija: ${res2}\n`);

// 3. Keshdan qayta o'qish tezligi
console.time("3. Keshdan Qayta O'qish (0ms bo'lishi kerak)");
const res3 = memoizedFibonacci(N);
console.timeEnd("3. Keshdan Qayta O'qish (0ms bo'lishi kerak)");
console.log(`Natija: ${res3}\n`);
// --- Counter Test ---
const counter = createCounter({ step: 5, min: -10, max: 12 });
console.log("Counter +5:", counter.increment()); // 5
console.log("Counter +5:", counter.increment()); // 10
console.log("Counter +5 (Limit 12):", counter.increment()); // 10 (Chunki keyingi qadam 15 > max)
console.log("Counter -5:", counter.decrement()); // 5

// --- Once Test ---
const processPayment = once((amount) => {
  console.log(`To'lov bajarildi: ${amount} UZS`);
  return "SUCCESS_TOKEN";
});

console.log(processPayment(50000)); // Konsolga yozuv chiqadi va token qaytadi
console.log(processPayment(50000)); // Faqat saqlangan token qaytadi, log ishlamaydi

// --- Debounce Test ---
const searchInput = debounce((text) => console.log(`API so'rov: ${text}`), 300);
searchInput("A");
searchInput("Ab");
searchInput("Abdu"); // 300ms kutiladi va faqat oxirgi chaqiriq konsolga chiqadi
