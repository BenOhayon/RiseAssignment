# Rise Home Assignment

This is a solution for Rise home assignment.  
The following script is implemented as an IIFE (**I**mmediately **I**nvoked **F**unction **E**xpression) that accepts 2 arguments:  

1. `elementId` - The ID of the div element in which the script will inject its logic.
2. `forecastDaysCount` - The number of forecast days to request weather data for. The default is 14 days as originally requested in the assignment.

The script creates a div with an input and a button.  
This div is then injected to the element whose ID is given as a parameter (`elementId`), or in the body if there's no such element.  

The user can enter a location coordinates or a location name (could be a state or a city). When clicking on the button, a request for the next `forecastDaysCount` days is sent to VisualCrossing weather API. The script then calculates for each day the average temperature for the next `forecastDaysCount` days.

Finally, the result is displayed under the input and the button elements as an unordered list as follows:  
- [Day]: [Average temperature]

For example:  
- Sunday: 25°C
- Monday: 27°C
- Tuesday: 30°C
- Wednesday: 21°C
- Thursday: 20°C
- Friday: 29°C
- Saturday: 19°C

The following is the script to inject:  

```js
<script>
    window.onload = () => {
        (function(elementId, forecastDaysCount = 14) {
            let selectedElement = document.querySelector(`#${elementId}`)
            if (!selectedElement) {
                selectedElement = document.querySelector('body')
            }

            // create the weather div
            const weatherDiv = document.createElement('div')
            const input = document.createElement('input')
            input.setAttribute('type', 'text')
            const button = document.createElement('button')
            button.textContent = 'Send'
            const resultDiv = document.createElement('div')
            weatherDiv.appendChild(input)
            weatherDiv.appendChild(button)
            weatherDiv.appendChild(resultDiv)
            selectedElement.appendChild(weatherDiv)
            
            // assign change listener to the button and the input
            button.addEventListener('click', () => {
                if (input.value.length === 0) return

                // fetch weather data
                const rawStartDate = new Date()
                const rawEndDate = new Date(rawStartDate)
                rawEndDate.setDate(rawStartDate.getDate() + (forecastDaysCount - 1))

                const formattedStartDate = `${rawStartDate.getFullYear()}-${rawStartDate.getMonth() + 1}-${rawStartDate.getDate()}`
                const formattedEndDate = `${rawEndDate.getFullYear()}-${rawEndDate.getMonth() + 1}-${rawEndDate.getDate()}`
            
                const url = `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${input.value}/${formattedStartDate}/${formattedEndDate}?unitGroup=metric&include=days&key=G4U2BHH6T7LQUCBTXFEETQP75&contentType=json`
                const options = {
                    "method": "GET",
                    "headers": {}
                }

                fetch(url, options)
                    .then(response => response.json())
                    .then(weatherData => {
                        const hasLessThanWeek = weatherData.days.length < 7
                        const resultArrayLength = hasLessThanWeek ? weatherData.days.length : 7
                        const result = new Array(resultArrayLength).fill(0)
                        
                        // handle weather data
                        for (let i = 0 ; i < resultArrayLength ; i++) {
                            let j = i, sum = 0, count = 0
                            const date = new Date(weatherData.days[i].datetime)
                            while (j < weatherData?.days?.length) {
                                sum += weatherData?.days[j]?.temp
                                count++
                                j += 7
                            }

                            result[i] = [date, Math.floor(sum / count)]
                        }

                        const resultUl = document.createElement('ul')
                        resultDiv.appendChild(resultUl)
                        result.forEach(([date, averageTemp]) => {
                            const li = document.createElement('li')
                            li.textContent = `${new Intl.DateTimeFormat('en-US', { weekday: 'long' }).format(date)}: ${averageTemp}°C`
                            resultUl.appendChild(li)
                        })
                        resultDiv.replaceChildren(resultUl)
                    })
                    .catch(err => {
                        resultDiv.replaceChildren("Something went wrong with the request")
                    });
            })
    })('weather')
}
</script>
```