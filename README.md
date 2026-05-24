const HaxballJS = require('haxball.js').default;
const axios = require('axios');

// Funkcja automatycznie pobierająca działające darmowe proxy z Hiszpanii
async function getSpainProxy() {
    try {
        console.log("Pobieranie darmowej listy proxy z Hiszpanii...");
        // API pobiera najnowsze darmowe proxy HTTP z lokalizacją w Hiszpanii (ES)
        const response = await axios.get('https://geonode.com');
        
        if (response.data && response.data.data && response.data.data.length > 0) {
            const proxy = response.data.data[0];
            const proxyUrl = `http://${proxy.ip}:${proxy.port}`;
            console.log(`Pomyślnie podpięto pod darmowe proxy: ${proxyUrl}`);
            return proxyUrl;
        }
        throw new Error("Brak wolnych proxy z Hiszpanii");
    } catch (error) {
        console.log("Darmowe listy proxy są przeciążone. Odpalam bezpośrednio...");
        return null;
    }
}

async function startRoom() {
    const proxyAddress = await getSpainProxy();
    const config = {};
    
    // Jeśli skrypt znalazł darmowe proxy, aplikuje je do konfiguracji
    if (proxyAddress) {
        config.proxy = proxyAddress;
    }

    HaxballJS(config).then((HBInit) => {
        const room = HBInit({
            roomName: "⚽ [ESP] Darmowe Proxy 2026 ⚽",
            maxPlayers: 16,
            public: true,
            noPlayer: true,
            token: "TUTAJ_WLEJ_SWOJE_NOWE_TOKEN_HEADLESS", // <--- Tutaj szybko wklejasz token!
            geo: { 
                code: "es", 
                lat: 40.4167, 
                lon: -3.7037 
            }
        });

        room.onRoomLink = (link) => {
            console.log("\n=================================");
            console.log("SUKCES! POKÓJ PREZ PROXY URUCHOMIONY!");
            console.log("LINK DO POKOJU: " + link);
            console.log("=================================\n");
        };

        // Automatyczny restart i szukanie nowego proxy, jeśli aktualne darmowe padnie
        room.onRoomClose = () => {
            console.log("Połączenie proxy przerwane. Szukam nowego darmowego IP za 5 sekund...");
            setTimeout(startRoom, 5000);
        };
    }).catch(err => {
        console.log("To proxy z Hiszpanii odrzuciło połączenie. Próbuję z kolejnym...");
        setTimeout(startRoom, 2000);
    });
}

startRoom();
