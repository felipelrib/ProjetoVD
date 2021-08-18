---
layout: page
title: Produtoras melhor avaliadas
---
<script type="text/javascript">
    function updateGraph() {
        if (!window.moviesByProductionCompany) {
            window.moviesByProductionCompany = {};
            moviesData.forEach(movie => {
                if (Array.isArray(movie.production_companies) && movie.vote_average) {
                    movie.production_companies.forEach(productionCompany => {
                        if (productionCompany.name) {
                            if (!moviesByProductionCompany[productionCompany.name]) {
                                moviesByProductionCompany[productionCompany.name] = [];
                            }
                            moviesByProductionCompany[productionCompany.name].push(movie);
                        }
                    });
                }
            });
            window.averageRatingByProductionCompany = {};
            for (const companyName in moviesByProductionCompany) {
                const ratingSum = moviesByProductionCompany[companyName].reduce((sum, movie) => {
                    return sum + movie.vote_average;
                }, 0);
                averageRatingByProductionCompany[companyName] = ratingSum / moviesByProductionCompany[companyName].length;
            }
        }
        chartData = [];
        for (const companyName in moviesByProductionCompany) {
            if (moviesByProductionCompany[companyName].length >= (filters.minimumMoviesAmount || 100)) {
                chartData.push([companyName, averageRatingByProductionCompany[companyName]]);
            }
        }

        chartData.sort((a, b) => b[1] - a[1]);
        chartData = chartData.slice(0, filters.resultsAmount || 10);

        Highcharts.chart('container', {
            chart: {
                type: 'column'
            },
            title: {
                text: 'Produtoras melhor avaliadas'
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
                    text: 'Nota média dos filmes'
                }
            },
            legend: {
                enabled: false
            },
            tooltip: {
                pointFormat: '{series.name}: <b>{point.y:.2f}</b>'
            },
            credits: {
                enabled: false
            },
            series: [{
                name: 'Nota média dos filmes',
                data: chartData,
                dataLabels: {
                    enabled: true,
                    rotation: -90,
                    color: '#FFFFFF',
                    align: 'right',
                    format: '{point.y:.2f}',
                    y: 10,
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

        if (filters.minimumMoviesAmount && isNaN(filters.minimumMoviesAmount)) {
            reportError("minimumMoviesAmount", "Por favor, insira um número válido.");
        } else if (filters.resultsAmount && isNaN(filters.resultsAmount)) {
            reportError("resultsAmount", "Por favor, insira um número válido.");
        } else {
            updateGraph();
        }
    }

    loadMoviesData().then(_ => {
        updateGraph();
        addListener("minimumMoviesAmount", validateInputChange);
        addListener("resultsAmount", validateInputChange);
    });
</script>
<p class="message">
A quantidade mínima de filmes deve ser definida para evitar outliers de produtoras que tenham poucos filmes bem-avaliados. A quantidade mínima de filmes para uma produtora ser considerada pode ser alterada no filtro abaixo.
</p>
<label for="minimumMoviesAmount">Quantidade mínima de filmes da produtora:</label>
<input type="number" id="minimumMoviesAmount" name="minimumMoviesAmount" value="100">
<br />
<label for="resultsAmount">Quantidade de produtoras a serem exibidas no gráfico:</label>
<input type="number" id="resultsAmount" name="resultsAmount" value="10">
<div id="container"></div>
<p class="message">
Todas elas são produtoras bastante conhecidas, sendo a Toho com a maior média. É uma produtora oriental que pode não ser bem conhecida à primeira vista, mas foi responsável por diversos filmes do gênero de animes e de filmes clássicos como o Godzilla. Além disso, há a presença da francesa France 2 Cinéma, mais focada em filmes “cult”.
</p>
