# weatherapp
import React, { useState, useEffect, createContext, useContext } from 'react';
import axios from 'axios';

// create context for storing app state
const WeatherContext = createContext();

function App() {
  // set up state for user's location and favorite locations
  const [location, setLocation] = useState(null);
  const [favorites, setFavorites] = useState([]);

  // get user's location on page load
  useEffect(() => {
    navigator.geolocation.getCurrentPosition(position => {
      setLocation({
        lat: position.coords.latitude,
        lon: position.coords.longitude
      });
    });
  }, []);

  // handle adding and removing favorite locations
  function addFavorite(newFavorite) {
    setFavorites([...favorites, newFavorite]);
  }

  function removeFavorite(favoriteToRemove) {
    setFavorites(favorites.filter(favorite => favorite.id !== favoriteToRemove.id));
  }

  // render app with WeatherContext provider
  return (
    <WeatherContext.Provider value={{ location, favorites, addFavorite, removeFavorite }}>
      <div className="App">
        <HomeScreen />
        <SearchScreen />
      </div>
    </WeatherContext.Provider>
  );
}

function HomeScreen() {
  const { location, favorites } = useContext(WeatherContext);
  const [weatherData, setWeatherData] = useState(null);

  // get current weather data for user's location
  useEffect(() => {
    if (location) {
      axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${location.lat}&lon=${location.lon}&appid=${process.env.REACT_APP_OPENWEATHERMAP_API_KEY}`)
        .then(response => {
          setWeatherData(response.data);
        })
        .catch(error => {
          console.log(error);
        });
    }
  }, [location]);

  // render current weather and favorite locations
  return (
    <div>
      {weatherData &&
        <div>
          <h2>Current Weather for {weatherData.name}</h2>
          <p>{weatherData.weather[0].main} ({weatherData.weather[0].description})</p>
          <p>Temperature: {Math.round((weatherData.main.temp - 273.15) * 9/5 + 32)}°F</p>
          <p>Humidity: {weatherData.main.humidity}%</p>
          <p>Wind: {Math.round(weatherData.wind.speed * 2.237)} mph</p>
        </div>
      }
      <h2>Favorite Locations</h2>
      {favorites.length > 0 ?
        <ul>
          {favorites.map(favorite => (
            <li key={favorite.id}>
              {favorite.name}
              <button onClick={() => removeFavorite(favorite)}>Remove</button>
            </li>
          ))}
        </ul>
        :
        <p>You have no favorite locations.</p>
      }
    </div>
  );
}

function SearchScreen() {
  const { addFavorite } = useContext(WeatherContext);
  const [searchQuery, setSearchQuery] = useState('');
  const [searchResults, setSearchResults] = useState([]);

  // search for locations based on user input
  useEffect(() => {
    if (searchQuery) {
      axios.get(`https://api.openweathermap.org/data/2.5/find?q=${searchQuery}&type=like&appid=${process.env.REACT_APP_OPENWEATHERMAP_API_KEY}`)
        .then(response => {
          setSearchResults(response.data.list);
        })
        .
