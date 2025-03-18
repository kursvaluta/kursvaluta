<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Premium Курсы Валют</title>
    <style>
        :root {
            --primary: #2a2a72;
            --secondary: #009ffd;
            --gradient: linear-gradient(45deg, #2a2a72, #009ffd);
            --background: #f8f9fa;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', sans-serif;
        }

        body {
            background: var(--background);
            min-height: 100vh;
            padding: 2rem;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
        }

        header {
            text-align: center;
            margin-bottom: 2.5rem;
            padding: 2rem;
            background: var(--gradient);
            border-radius: 15px;
            color: white;
            box-shadow: 0 10px 20px rgba(0,0,0,0.1);
        }

        .converter {
            background: white;
            border-radius: 15px;
            padding: 2rem;
            margin-bottom: 2rem;
            box-shadow: 0 5px 15px rgba(0,0,0,0.05);
        }

        .converter-grid {
            display: grid;
            grid-template-columns: 1fr auto 1fr;
            gap: 1rem;
            align-items: center;
        }

        input, select {
            padding: 1rem;
            border: 2px solid #e0e0e0;
            border-radius: 10px;
            font-size: 1.1rem;
            width: 100%;
            transition: all 0.3s ease;
        }

        input:focus, select:focus {
            border-color: var(--secondary);
            outline: none;
        }

        .rates-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 1.5rem;
            margin-top: 2rem;
        }

        .rate-card {
            background: white;
            padding: 1.5rem;
            border-radius: 15px;
            text-align: center;
            box-shadow: 0 5px 15px rgba(0,0,0,0.05);
            transition: transform 0.3s ease;
        }

        .rate-card:hover {
            transform: translateY(-5px);
        }

        .currency-name {
            color: var(--primary);
            font-size: 1.5rem;
            margin-bottom: 0.5rem;
        }

        .currency-rate {
            color: var(--secondary);
            font-size: 2rem;
            font-weight: bold;
        }

        footer {
            text-align: center;
            margin-top: 3rem;
            color: #666;
        }

        @media (max-width: 768px) {
            .converter-grid {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>Премиум Курсы Валют</h1>
            <p id="updateTime"></p>
        </header>

        <div class="converter">
            <div class="converter-grid">
                <input type="number" id="amount" placeholder="Сумма" value="1000">
                <select id="fromCurrency" onchange="convert()">
                    <option value="USD">USD</option>
                    <option value="EUR">EUR</option>
                    <option value="USDT">USDT</option>
                    <option value="RUB">RUB</option>
                </select>
                <select id="toCurrency" onchange="convert()">
                    <option value="RUB">RUB</option>
                    <option value="USD">USD</option>
                    <option value="EUR">EUR</option>
                    <option value="USDT">USDT</option>
                </select>
                <div id="result" class="currency-rate">0.00</div>
            </div>
        </div>

        <div class="rates-grid" id="ratesGrid">
            <div class="rate-card">
                <div class="currency-name">Доллар (USD)</div>
                <div class="currency-rate" id="usdRate">...</div>
            </div>
            <div class="rate-card">
                <div class="currency-name">Евро (EUR)</div>
                <div class="currency-rate" id="eurRate">...</div>
            </div>
            <div class="rate-card">
                <div class="currency-name">Tether (USDT)</div>
                <div class="currency-rate" id="usdtRate">...</div>
            </div>
        </div>

        <footer>
            <p>© 2024 Premium Exchange. Все курсы защищены от изменений.</p>
        </footer>
    </div>

    <script>
        (() => {
            const CORRECTION = 1.3;
            const USDT_RATE = 95.5; // Фиксированный курс USDT
            
            let rates = new Proxy({}, {
                set() {
                    return false; // Блокировка изменений
                }
            });

            async function loadRates() {
                try {
                    const response = await fetch('https://api.exchangerate-api.com/v4/latest/RUB');
                    const data = await response.json();
                    
                    // Расчет защищенных курсов
                    const securedUSD = (1 / data.rates.USD + CORRECTION).toFixed(2);
                    const securedEUR = (1 / data.rates.EUR + CORRECTION).toFixed(2);
                    
                    // Обновление интерфейса
                    document.getElementById('usdRate').textContent = `${securedUSD} ₽`;
                    document.getElementById('eurRate').textContent = `${securedEUR} ₽`;
                    document.getElementById('usdtRate').textContent = `${USDT_RATE} ₽`;
                    
                    // Обновление времени
                    document.getElementById('updateTime').textContent = 
                        `Обновлено: ${new Date().toLocaleString('ru-RU')}`;
                    
                    // Сохранение в защищенный объект
                    Object.defineProperty(rates, 'USD', { value: securedUSD });
                    Object.defineProperty(rates, 'EUR', { value: securedEUR });
                    Object.defineProperty(rates, 'USDT', { value: USDT_RATE });
                    Object.defineProperty(rates, 'RUB', { value: 1 });
                    
                    convert();
                } catch (error) {
                    console.error('Ошибка загрузки:', error);
                    document.getElementById('ratesGrid').innerHTML = 
                        '<div class="rate-card">Ошибка обновления данных</div>';
                }
            }

            function convert() {
                const amount = parseFloat(document.getElementById('amount').value) || 0;
                const from = document.getElementById('fromCurrency').value;
                const to = document.getElementById('toCurrency').value;
                
                const result = (amount * rates[from]) / rates[to];
				const ADMIN_PASSWORD = "MySecretPass123"; // Замените на свой пароль
                document.getElementById('result').textContent = result.toFixed(2);
            }

            // Защита от изменений
            Object.freeze(Object.prototype);
            document.addEventListener('keydown', e => {
                if (e.ctrlKey || e.metaKey) e.preventDefault();
            });
            document.addEventListener('contextmenu', e => e.preventDefault());

            // Инициализация
            loadRates();
            setInterval(loadRates, 300000);
            document.getElementById('amount').addEventListener('input', convert);
        })();
    </script>
</body>
</html>
