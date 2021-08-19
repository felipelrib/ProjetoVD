---
layout: page
title: Média de bilheteria por mês de lançamento
---
<script type="text/javascript">
    Highcharts.setOptions({
        lang: {
            numericSymbols: [" mil", " milhões", " bilhões", " trilhões"]
        }
    });

    function updateGraph() {
        if (!filters.startYear) {
            filters.startYear = 2010;
        }
        if (!filters.endYear) {
            filters.endYear = 2013;
        }
        const startYear = filters.startYear;
        const endYear = filters.endYear;
        let chartData = []
        for (let i = 0; i < endYear - startYear + 1; i++) {
            chartData.push({
                name: startYear + i,
                data: Array(12).fill(0)
            });
        }
        chartData = moviesData.reduce((chartData, row) => {
            const revenue = parseFloat(row.revenue);
            if (row.release_date && !isNaN(revenue)) {
                const releaseDate = new Date(row.release_date);
                if (releaseDate instanceof Date && isFinite(releaseDate)) {
                    // Tries to fix a problem due to the date being created in UTC timezone.
                    releaseDate.setHours(releaseDate.getHours() + 12);
                    if (releaseDate.getFullYear() >= startYear && releaseDate.getFullYear() <= endYear) {
                        chartData[releaseDate.getFullYear() - startYear].data[releaseDate.getMonth()] += revenue;
                    }
                }
            }
            return chartData;
        }, chartData);
        Highcharts.chart('container', {
            title: {
                text: 'Média de bilheteria por mês de lançamento'
            },
            xAxis: {
                categories: ['Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho', 'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro']
            },
            yAxis: {
                title: {
                    text: 'Valor de bilheteria (USD)'
                }
            },
            legend: {
                layout: 'vertical',
                align: 'right',
                verticalAlign: 'middle'
            },
            credits: {
                enabled: false
            },
            series: chartData
        });
    }

    function validateInputChange() {
        function reportError(inputName, errorMessage) {
            let input = document.querySelector(`input[name=${inputName}]`);
            input.setCustomValidity(errorMessage);
            input.reportValidity();
        }

        if (!filters.startYear || isNaN(filters.startYear)) {
            reportError("startYear", "Por favor, insira um número válido.");
        } else if (!filters.endYear || isNaN(filters.endYear)) {
            reportError("endYear", "Por favor, insira um número válido.");
        } else if (filters.startYear > filters.endYear) {
            reportError("startYear", "O ano inicial não pode ser maior que o ano final.");
        } else if (filters.endYear - filters.startYear > 20) {
            reportError("startYear", "A diferença entre o ano inicial e o ano final não pode ser maior que 20.");
        } else {
            updateGraph();
        }
    }

    loadMoviesData().then(_ => {
        updateGraph();
        addListener("startYear", validateInputChange);
        addListener("endYear", validateInputChange);
    });
</script>
<label for="startYear">Ano de lançamento inicial:</label>
<input type="number" id="startYear" name="startYear" value="2010">
<br />
<label for="endYear">Ano de lançamento final:</label>
<input type="number" id="endYear" name="endYear" value="2013">
<div id="container"></div>
<p class="message">
Podemos ver uma tendência dos meses no meio e no final do ano a terem as maiores receitas. Isso pode ser explicado pelo fato da aproximação do lançamento nos cinemas com as datas comuns para as férias, ficando os cinemas lotados.
</p>
