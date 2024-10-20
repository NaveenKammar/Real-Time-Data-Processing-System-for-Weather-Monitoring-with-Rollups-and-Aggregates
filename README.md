# Real-Time-Data-Processing-System-for-Weather-Monitoring-with-Rollups-and-Aggregates
It processes real-time weather information such as temperature, perceived temperature, and main weather conditions, converting temperatures from Kelvin to Celsius based on user preferences. The system stores daily weather summaries, including average, maximum, and minimum temperatures, and tracks dominant weather conditions.

for Project:
Backend:Java,Springboot,Hibernate(Thymleaf)
frontend:Html,Css,Javascript
Database:Mysql

Project Structure:
- src
  - main
    - java
      - com.example.weather
        - controller
          - WeatherController.java
        - service
          - WeatherService.java
          - OpenWeatherMapService.java
        - model
          - WeatherData.java
        - repository
          - WeatherRepository.java
    - resources
      - templates
        - weatherSummary.html
      - application.properties
  - test
    - java
      - com.example.weather
        - WeatherServiceTests.java

Backend Code
1.WeatherData.java

package com.example.weather.model;

import javax.persistence.*;
import java.time.LocalDate;

@Entity
@Table(name = "weather_data")
public class WeatherData {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String cityName;
    private double temp;
    private double feelsLike;
    private LocalDate date;
    private String mainWeather;
    private double maxTemp;
    private double minTemp;
	public Long getId() {
		return id;
	}
	public void setId(Long id) {
		this.id = id;
	}
	public String getCityName() {
		return cityName;
	}
	public void setCityName(String cityName) {
		this.cityName = cityName;
	}
	public double getTemp() {
		return temp;
	}
	public void setTemp(double temp) {
		this.temp = temp;
	}
	public double getFeelsLike() {
		return feelsLike;
	}
	public void setFeelsLike(double feelsLike) {
		this.feelsLike = feelsLike;
	}
	public LocalDate getDate() {
		return date;
	}
	public void setDate(LocalDate date) {
		this.date = date;
	}
	public String getMainWeather() {
		return mainWeather;
	}
	public void setMainWeather(String mainWeather) {
		this.mainWeather = mainWeather;
	}
	public double getMaxTemp() {
		return maxTemp;
	}
	public void setMaxTemp(double maxTemp) {
		this.maxTemp = maxTemp;
	}
	public double getMinTemp() {
		return minTemp;
	}
	public void setMinTemp(double minTemp) {
		this.minTemp = minTemp;
	}
}

2.WeatherRepository.java
package com.example.weather.repository;

import com.example.weather.model.WeatherData;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface WeatherRepository extends JpaRepository<WeatherData, Long> {

}

3. OpenWeatherMapService.java
package com.example.weather.service;

import com.example.weather.model.WeatherData;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;
import org.json.JSONObject;

import java.time.Instant;
import java.time.LocalDate;
import java.time.ZoneId;

@Service
public class OpenWeatherMapService {
    private final String apiKey = "your_openweathermap_api_key";
    private final String urlTemplate = "https://api.openweathermap.org/data/2.5/weather?q=%s&appid=" + apiKey;

    public WeatherData getWeatherData(String city) {
        String url = String.format(urlTemplate, city);
        RestTemplate restTemplate = new RestTemplate();
        String result = restTemplate.getForObject(url, String.class);
        return parseWeatherData(result, city);
    }

    private WeatherData parseWeatherData(String response, String city) {
        JSONObject json = new JSONObject(response);

        WeatherData weatherData = new WeatherData();
        weatherData.setCityName(city);
        weatherData.setTemp(json.getJSONObject("main").getDouble("temp") - 273.15); // Kelvin to Celsius
        weatherData.setFeelsLike(json.getJSONObject("main").getDouble("feels_like") - 273.15);
        weatherData.setMainWeather(json.getJSONArray("weather").getJSONObject(0).getString("main"));
        weatherData.setDate(Instant.ofEpochSecond(json.getLong("dt"))
                          .atZone(ZoneId.systemDefault()).toLocalDate());

        weatherData.setMaxTemp(json.getJSONObject("main").getDouble("temp_max") - 273.15);
        weatherData.setMinTemp(json.getJSONObject("main").getDouble("temp_min") - 273.15);

        return weatherData;
    }
}


4. WeatherService.java
   package com.example.weather.service;

import com.example.weather.model.WeatherData;
import com.example.weather.repository.WeatherRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class WeatherService {

    @Autowired
    private WeatherRepository weatherRepository;

    @Autowired
    private OpenWeatherMapService openWeatherMapService;

    public WeatherData fetchAndSaveWeather(String city) {
        WeatherData weatherData = openWeatherMapService.getWeatherData(city);
        return weatherRepository.save(weatherData);
    }

    public List<WeatherData> getDailyWeatherSummary() {
        return weatherRepository.findAll();
    }

    public void checkAlertThreshold(double threshold) {
        // Logic to check if temperature exceeds threshold and alert
    }
}

5.WeatherController.java
package com.example.weather.controller;

import com.example.weather.service.WeatherService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class WeatherController {

    @Autowired
    private WeatherService weatherService;

    @GetMapping("/weather")
    public String getWeatherSummary(Model model) {
        model.addAttribute("weatherData", weatherService.getDailyWeatherSummary());
        return "weatherSummary";
    }

    @GetMapping("/fetchWeather")
    public String fetchWeather(Model model) {
        weatherService.fetchAndSaveWeather("Delhi");
        weatherService.fetchAndSaveWeather("Mumbai");
        weatherService.fetchAndSaveWeather("Chennai");
        weatherService.fetchAndSaveWeather("Bangalore");
        weatherService.fetchAndSaveWeather("Kolkata");
        weatherService.fetchAndSaveWeather("Hyderabad");
        return "redirect:/weather";
    }
}

6. Unit Test: WeatherServiceTests.java
package com.example.weather;

import com.example.weather.service.WeatherService;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class WeatherServiceTests {

    @Test
    void testTemperatureConversion() {
        // Add the temperature conversion method to WeatherService
        WeatherService service = new WeatherService();
        double kelvin = 300;
        double celsius = service.convertKelvinToCelsius(kelvin);
        assertEquals(26.85, celsius, 0.1);
    }

    @Test
    void testFetchWeatherData() {
        // Add test for fetching weather data and saving it
    }
}


7.weatherSummary.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Weather Summary</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>Daily Weather Summary</h1>
    <table>
        <thead>
            <tr>
                <th>City</th>
                <th>Temperature (°C)</th>
                <th>Feels Like (°C)</th>
                <th>Main Weather</th>
                <th>Date</th>
            </tr>
        </thead>
        <tbody>
            <tr th:each="weather : ${weatherData}">
                <td th:text="${weather.cityName}"></td>
                <td th:text="${weather.temp}"></td>
                <td th:text="${weather.feelsLike}"></td>
                <td th:text="${weather.mainWeather}"></td>
                <td th:text="${weather.date}"></td>
            </tr>
        </tbody>
    </table>
</body>
</html>

8.styles.css
table {
    width: 100%;
    border-collapse: collapse;
}

th, td {
    padding: 8px;
    text-align: left;
    border-bottom: 1px solid #ddd;
}

th {
    background-color: #f2f2f2;
}

9. Database Configuration: application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/weather_db
spring.datasource.username=root
spring.datasource.password=NaveenK
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true


   



