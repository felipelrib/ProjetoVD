---
layout: page
title: Data de lançamento de um filme
---
<script type="text/javascript">
    function updateGraph() {
        const chartData = moviesData.reduce((chartData, row) => {
            if (row.release_date) {
                const releaseDate = new Date(row.release_date);
                if (releaseDate instanceof Date && isFinite(releaseDate)) {
                    // Tries to fix a problem due to the date being created in UTC timezone.
                    releaseDate.setHours(releaseDate.getHours() + 12);
                    if ((!filters.startYear || releaseDate.getFullYear() >= filters.startYear) && (!filters.endYear || releaseDate.getFullYear() <= filters.endYear)) {
                        chartData[releaseDate.getDay()][1]++;
                    }
                }
            }
            return chartData;
            }, [
            ["Domingo", 0],
            ["Segunda", 0],
            ["Terça", 0],
            ["Quarta", 0],
            ["Quinta", 0],
            ["Sexta", 0],
            ["Sábado", 0]
        ]);
        const totalValidDates = chartData.reduce((sum, day) => sum + day[1], 0);
        // Get the percentage of each day.
        chartData.forEach(day => day[1] = day[1] * 100 / totalValidDates);

        Highcharts.chart('container', {
            chart: {
                type: 'column'
            },
            title: {
                text: 'Dia do lançamento de um filme'
            },
            xAxis: {
                type: 'category',
                labels: {
                    style: {
                        fontSize: '13px',
                        fontFamily: 'Verdana, sans-serif'
                    }
                }
            },
            yAxis: {
                min: 0,
                title: {
                    text: 'Lançamentos no dia (%)'
                }
            },
            legend: {
                enabled: false
            },
            tooltip: {
                pointFormat: '{series.name}: <b>{point.y:.1f}%</b>'
            },
            credits: {
                enabled: false
            },
            series: [{
                name: 'Porcentagem de filmes lançados no dia',
                data: chartData,
                dataLabels: {
                    enabled: true,
                    rotation: -90,
                    color: '#FFFFFF',
                    align: 'right',
                    format: '{point.y:.1f}', // one decimal
                    y: 10, // 10 pixels down from the top
                    style: {
                        fontSize: '13px',
                        fontFamily: 'Verdana, sans-serif'
                    }
                }
            }]
        });
    }

    function validateInputChange() {
        function reportError(inputName, errorMessage) {
            let input = document.querySelector(`input[name=${inputName}]`);
            input.setCustomValidity(errorMessage);
            input.reportValidity();
        }

        if (filters.startYear && isNaN(filters.startYear)) {
            reportError("startYear", "Por favor, insira um número válido.");
        } else if (filters.endYear && isNaN(filters.endYear)) {
            reportError("endYear", "Por favor, insira um número válido.");
        } else if (filters.startYear && filters.endYear && filters.startYear > filters.endYear) {
            reportError("startYear", "O ano inicial não pode ser maior que o ano final.");
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
<input type="number" id="startYear" name="startYear" value="1874">
<br />
<label for="endYear">Ano de lançamento final:</label>
<input type="number" id="endYear" name="endYear" value="2020">
<div id="container"></div>
<p class="message">
Verificamos, então, que o dia mais comum para lançamentos de filmes no mundo é na sexta-feira. Outro fato interessante é que o segundo e o terceiro dias são, respectivamente, quinta-feira e quarta-feira, indicando uma tendência ao lançamento em dias entre o meio e o final da semana útil, embora pudesse ser imaginado intuitivamente que os filmes seriam lançados mais frequentemente no final de semana, fora de dias úteis.
</p>
