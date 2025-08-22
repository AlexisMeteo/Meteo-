<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Météo Lombard (Doubs)</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f0f4f8;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }
    .container {
      background: white;
      padding: 20px 30px;
      border-radius: 15px;
      box-shadow: 0px 4px 12px rgba(0,0,0,0.1);
      text-align: center;
      max-width: 400px;
    }
    h1 {
      color: #0077b6;
      font-size: 22px;
      margin-bottom: 15px;
    }
    #meteo {
      font-size: 18px;
      color: #333;
      line-height: 1.5;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Météo Lombard (Doubs)</h1>
    <div id="meteo">Chargement...</div>
  </div>

  <script>
    const apiKey = "9e9ba5c3a9044b4b6019dcb0acd60e3"; // Remplacez par votre clé OpenWeather
    const ville = "Lombard,FR";
    const url = `https://api.openweathermap.org/data/2.5/forecast?q=${ville}&units=metric&lang=fr&appid=${apiKey}`;

    fetch(url)
      .then(response => response.json())
      .then(data => {
        // Regrouper les prévisions par jour
        const jours = {};
        data.list.forEach(item => {
          const date = new Date(item.dt * 1000);
          const jour = date.toLocaleDateString("fr-FR", { weekday: 'long', day: 'numeric', month: 'long' });

          if (!jours[jour]) {
            jours[jour] = { min: item.main.temp, max: item.main.temp, descriptions: [] };
          }
          jours[jour].min = Math.min(jours[jour].min, item.main.temp);
          jours[jour].max = Math.max(jours[jour].max, item.main.temp);
          jours[jour].descriptions.push(item.weather[0].description);
        });

        const joursCles = Object.keys(jours);
        const today = jours[joursCles[0]];
        const tomorrow = jours[joursCles[1]];

        function resumeDescription(descs) {
          const freq = {};
          descs.forEach(d => freq[d] = (freq[d] || 0) + 1);
          return Object.keys(freq).reduce((a, b) => freq[a] > freq[b] ? a : b);
        }

        const todayDesc = resumeDescription(today.descriptions);
        const tomorrowDesc = resumeDescription(tomorrow.descriptions);

        const meteoTexte = 
          `Aujourd'hui (${joursCles[0]}) à Lombard : minimum ${Math.round(today.min)}°C, maximum ${Math.round(today.max)}°C, ${todayDesc}. 
           Demain (${joursCles[1]}) : minimum ${Math.round(tomorrow.min)}°C, maximum ${Math.round(tomorrow.max)}°C, ${tomorrowDesc}.`;

        document.getElementById("meteo").innerText = meteoTexte;

        const synth = window.speechSynthesis;
        const utter = new SpeechSynthesisUtterance(meteoTexte);
        utter.lang = "fr-FR";
        synth.speak(utter);
      })
      .catch(err => {
        document.getElementById("meteo").innerText = "Impossible de récupérer la météo.";
        console.error(err);
      });
  </script>
</body>
</html>
