# Kilometer_Beregner
Bruger 2 kilometerstande fra 2 datoer og beregner det årlige kilometer forbrug
<!DOCTYPE html>
<html lang="da">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Årligt Kilometergennemsnit (GF-inspireret)</title>
    
    <!-- Tailwind CSS CDN for nem styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Sørger for, at appen fylder hele skærmen og bruger Inter-skrifttypen */
        body {
            font-family: 'Inter', sans-serif;
            /* Baggrundsbillede fra AI-genereret prompt, da vi ikke kan bruge de uploadede billeder direkte */
            background-image: url('https://image.pollinations.ai/prompt/A%20family%20of%20two%20adults%20and%20two%20children%20standing%20in%20front%20of%20a%20green%20car,%20watching%20a%20sunset.%20Realistic%2C%20cinematic%20lighting%2C%20warm%20tones.');
            background-size: cover;
            background-position: center;
            background-repeat: no-repeat;
            background-attachment: fixed; /* Billedet forbliver fast, når man scroller */
            position: relative; /* Nødvendig for overlay */
        }

        /* Overlay for bedre læsbarhed af tekst på baggrundsbilledet */
        body::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(0, 0, 0, 0.4); /* Mørkere overlay */
            z-index: -1; /* Placeres under indholdet */
        }

        /* Definerer GF-inspirerede farver */
        .gf-green-bg {
            background-color: #007a3c; /* GF Grøn */
            color: white;
        }
        .gf-text-green {
            color: #007a3c;
        }
        .gf-yellow-accent {
            background-color: #f9a626; /* GF Gul/Orange til knapper/fokus */
            color: #1a365d;
        }
        .input-style {
            padding: 10px;
            border: 1px solid #e2e8f0; 
            border-radius: 8px; /* Lidt mere afrundet end standard */
            width: 100%;
            transition: border-color 0.2s, box-shadow 0.2s;
        }
        .input-style:focus {
            outline: none;
            border-color: #f9a626; /* Gul/Orange ved fokus */
            box-shadow: 0 0 0 3px rgba(249, 166, 38, 0.3);
        }
        /* Style til at indikere et obligatorisk felt */
        .required-label::after {
            content: '*';
            color: #ef4444; /* Rød stjerne */
            margin-left: 2px;
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">

    <!-- Hovedcontainer -->
    <div class="w-full max-w-lg bg-white p-6 sm:p-10 rounded-xl shadow-xl border border-gray-100 relative z-10">
        <h1 class="text-3xl font-extrabold gf-text-green mb-4 text-center">
            Beregn dit kørselsbehov
        </h1>
        <p class="text-gray-600 mb-8 text-center">
            Få et hurtigt overblik over bilens årlige gennemsnitlige kørsel.
        </p>

        <!-- Input Formular -->
        <div id="calculatorForm" class="space-y-6">

            <!-- Fejlmeddelelse boks -->
            <div id="errorBox" class="p-3 bg-red-100 border border-red-400 text-red-700 rounded-xl hidden" role="alert">
                <!-- Fejlmeddelelser indsættes her -->
            </div>

            <!-- Måling 1 Gruppe -->
            <div class="bg-white p-5 rounded-xl border border-gray-200 shadow-md">
                <h2 class="text-xl font-semibold gf-text-green mb-4">Startdato og KM-stand</h2>
                <div>
                    <label for="date1" class="block text-sm font-medium text-gray-700 mb-1 required-label">Dato 1 (DDMMÅÅÅÅ)</label>
                    <input type="text" id="date1" placeholder="Indtast 8 cifre (f.eks. 01012023)" class="input-style" maxlength="10" aria-label="Start Dato">
                </div>
                <div class="mt-4">
                    <label for="mileage1" class="block text-sm font-medium text-gray-700 mb-1 required-label">Kilometertal 1 (km)</label>
                    <input type="number" id="mileage1" placeholder="Kilometerstand" class="input-style" min="0" aria-label="Start Kilometertal">
                </div>
            </div>

            <!-- Måling 2 Gruppe -->
            <div class="bg-white p-5 rounded-xl border border-gray-200 shadow-md">
                <h2 class="text-xl font-semibold gf-text-green mb-4">Slutdato og KM-stand</h2>
                <div>
                    <label for="date2" class="block text-sm font-medium text-gray-700 mb-1 required-label">Dato 2 (DDMMÅÅÅÅ)</label>
                    <input type="text" id="date2" placeholder="Indtast 8 cifre (f.eks. 01012024)" class="input-style" maxlength="10" aria-label="Slut Dato">
                </div>
                <div class="mt-4">
                    <label for="mileage2" class="block text-sm font-medium text-gray-700 mb-1 required-label">Kilometertal 2 (km)</label>
                    <input type="number" id="mileage2" placeholder="Kilometerstand" class="input-style" min="0" aria-label="Slut Kilometertal">
                </div>
            </div>

        </div>

        <!-- Resultatboks (Bruger GF Grøn farve for at fremhæve nøgleresultatet, ligesom en GF-boks) -->
        <div id="resultBox" class="mt-8 p-6 rounded-xl text-center shadow-2xl gf-green-bg border-4 border-white">
            <h3 class="text-xl font-semibold mb-2">Dit årlige gennemsnit er:</h3>
            <p id="annualAverage" class="text-5xl font-extrabold text-white my-4">--</p>
            <p class="mt-1 text-gray-200">kilometers gennemsnit per år</p>
            
            <hr class="my-4 border-green-200 opacity-50">
            
            <p id="totalDistance" class="text-md font-medium text-gray-100">Afstand: -- km</p>
            <p id="totalDays" class="text-md font-medium text-gray-100">Periode: -- dage</p>
        </div>

    </div>

    <script>
        // Opsætning af globale konstanter for UI-elementer
        const date1Input = document.getElementById('date1');
        const date2Input = document.getElementById('date2');
        const mileage1Input = document.getElementById('mileage1');
        const mileage2Input = document.getElementById('mileage2');
        const resultBox = document.getElementById('resultBox');
        const errorBox = document.getElementById('errorBox');
        const annualAverageDisplay = document.getElementById('annualAverage');
        const totalDistanceDisplay = document.getElementById('totalDistance');
        const totalDaysDisplay = document.getElementById('totalDays');
        
        /**
         * Konverterer en dato (DDMMÅÅÅÅ eller DD-MM-ÅÅÅÅ) til et JavaScript Date-objekt.
         * @param {string} dateString - Dato strengen.
         * @returns {Date | null} - Date objektet eller null ved fejl.
         */
        function parseDanishDate(dateString) {
            const digitsOnly = dateString.replace(/\D/g, ''); // Fjerner alle ikke-cifre
            
            if (digitsOnly.length !== 8) {
                return null;
            }

            // Udpakker dele: DD, MM, ÅÅÅÅ
            const day = parseInt(digitsOnly.substring(0, 2), 10);
            const month = parseInt(digitsOnly.substring(2, 4), 10);
            const year = parseInt(digitsOnly.substring(4, 8), 10);

            // Grundlæggende validering
            if (day === 0 || day > 31 || month === 0 || month > 12 || year < 1900) {
                return null;
            }

            // Opretter dato (Måned er 0-indekseret i JS: month - 1)
            const date = new Date(year, month - 1, day);
            
            // Yderligere validering for at fange ugyldige datoer som f.eks. 30. februar
            if (date.getFullYear() === year && date.getMonth() === month - 1 && date.getDate() === day) {
                // Returnerer datoen ved midnat UTC for at undgå tidszoneproblemer i beregningen
                return new Date(Date.UTC(year, month - 1, day));
            }
            return null;
        }

        /**
         * Indsætter automatisk bindestreger i datoinputtet (DD-MM-ÅÅÅÅ).
         * @param {Event} event - Input-begivenheden.
         */
        function autoFormatDate(event) {
            let input = event.target;
            let value = input.value.replace(/\D/g, ''); // Fjerner alle ikke-cifre
            let formattedValue = '';

            // Tager 2, 2 og 4 cifre
            if (value.length > 0) formattedValue += value.substring(0, 2);
            if (value.length >= 3) formattedValue += '-' + value.substring(2, 4);
            if (value.length >= 5) formattedValue += '-' + value.substring(4, 8);
            
            // Sikrer max 10 tegn (8 cifre + 2 bindestreger)
            input.value = formattedValue.substring(0, 10);
            
            // Trigger hovedberegningen efter formattering
            calculateAverageMileage();
        }

        /**
         * Viser en fejlmeddelelse i UI'en og nulstiller resultatet.
         * @param {string} message - Fejlmeddelelsen, der skal vises.
         */
        function displayError(message) {
            errorBox.textContent = message;
            errorBox.classList.remove('hidden');
            
            // Nulstil resultater og vis neutral farve/tekst
            annualAverageDisplay.textContent = 'Fejl';
            totalDistanceDisplay.textContent = 'Afstand: Ugyldig';
            totalDaysDisplay.textContent = 'Periode: Ugyldig';
            
            // Skift resultatboks til en neutral/fejl farve
            resultBox.classList.remove('gf-green-bg');
            resultBox.classList.add('bg-red-700'); /* Rød baggrund for fejl */
            resultBox.classList.add('text-white');
        }

        /**
         * Skjuler fejlmeddelelsesboksen og sætter resultatformat til succes.
         */
        function clearError() {
            errorBox.textContent = '';
            errorBox.classList.add('hidden');
            // Sæt resultatboks tilbage til GF Grøn
            resultBox.classList.add('gf-green-bg');
            resultBox.classList.remove('bg-red-700'); /* Fjern rød baggrund */
            resultBox.classList.remove('text-white'); // Sørg for at teksten er hvid i gf-green-bg
        }

        /**
         * Hovedfunktionen til at beregne det årlige kilometergennemsnit. Kører automatisk ved inputændring.
         */
        function calculateAverageMileage() {
            
            const date1Str = date1Input.value.trim();
            const date2Str = date2Input.value.trim();
            const mileage1 = parseFloat(mileage1Input.value);
            const mileage2 = parseFloat(mileage2Input.value);
            
            // 1. Kontroller for tomme felter
            if (!date1Str || !date2Str || isNaN(mileage1) || isNaN(mileage2)) {
                // Nulstil resultatet uden at vise en fejl, hvis felterne er tomme
                clearError(); 
                annualAverageDisplay.textContent = '--';
                totalDistanceDisplay.textContent = 'Afstand: -- km';
                totalDaysDisplay.textContent = 'Periode: -- dage';
                return; 
            }

            // 2. Parse og valider datoer
            const date1 = parseDanishDate(date1Str);
            const date2 = parseDanishDate(date2Str);

            if (!date1 || !date2) {
                return displayError("Datoformat er ugyldigt. Brug venligst 8 cifre (DDMMÅÅÅÅ).");
            }
            if (date2 <= date1) {
                return displayError("Slutdato (Dato 2) skal være efter Startdato (Dato 1).");
            }

            // 3. Valider kilometertal
            if (mileage2 < mileage1) {
                return displayError("Kilometertal 2 skal være højere end Kilometertal 1.");
            }
            if (mileage1 < 0 || mileage2 < 0) {
                return displayError("Kilometertal skal være positive værdier.");
            }


            // 4. Beregn
            const timeDifferenceMs = date2.getTime() - date1.getTime();
            const totalDays = timeDifferenceMs / (1000 * 60 * 60 * 24);
            const totalYears = totalDays / 365.25; // Inkluderer skudår
            const totalDistance = mileage2 - mileage1;

            if (totalYears === 0) {
                 return displayError("Perioden er for kort til at beregne et meningsfuldt gennemsnit.");
            }
            const annualAverage = totalDistance / totalYears;

            // 5. Vis resultat i UI
            clearError();
            annualAverageDisplay.textContent = annualAverage.toLocaleString('da-DK', {
                maximumFractionDigits: 0
            });
            totalDistanceDisplay.textContent = `Afstand: ${totalDistance.toLocaleString('da-DK')} km`;
            totalDaysDisplay.textContent = `Periode: ${totalDays.toLocaleString('da-DK', { maximumFractionDigits: 0 })} dage (ca. ${totalYears.toFixed(2)} år)`;
        }

        // --- Opsætning af Løbende Opdatering ---
        date1Input.addEventListener('input', autoFormatDate);
        date2Input.addEventListener('input', autoFormatDate);
        
        mileage1Input.addEventListener('input', calculateAverageMileage);
        mileage2Input.addEventListener('input', calculateAverageMileage);
        
        // --- Sæt Standardværdier (Eksempel) ---
        const today = new Date();
        const oneYearAgo = new Date();
        oneYearAgo.setFullYear(today.getFullYear() - 1);

        const formatDateTo8Digits = (date) => {
            const d = String(date.getDate()).padStart(2, '0');
            const m = String(date.getMonth() + 1).padStart(2, '0');
            const y = date.getFullYear();
            return `${d}${m}${y}`; // DDMMÅÅÅÅ format
        };

        // Indsæt værdier, som derefter formateres af autoFormatDate
        date2Input.value = formatDateTo8Digits(today);
        date1Input.value = formatDateTo8Digits(oneYearAgo);
        mileage1Input.value = 50000;
        mileage2Input.value = 75000;
        
        // Kør beregningen én gang med eksempelværdierne, når siden indlæses
        window.onload = function() {
            autoFormatDate({target: date1Input}); // Formaterer og trigger beregning for Dato 1
            autoFormatDate({target: date2Input}); // Formaterer og trigger beregning for Dato 2
        };

    </script>
</body>
</html>

