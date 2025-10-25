# Klient TODO w Vue.js

Prosta aplikacja typu TODO zbudowana w oparciu o **Vue.js**, umożliwiająca dodawanie, oznaczanie jako wykonane oraz usuwanie zadań danego dnia.

## Elementy do rozwoju projektu:
 - Loading animation podczas pobierania rekordów z api
 - Możliwość sortowania przez użytkownika
 - wyświetlenie widoku logowania dla użytkownika

## Uruchomienie projektu lokalnie

1. Sklonuj repozytorium:
   git clone https://github.com/RadoslawBarczynski/TodoClient
2. Utwórz plik w folderze todoAppClient o nazwie .env oraz dodaj tam:
VITE_API_URL={link localhost api}
3. Uruchom VS Code i wpisz w terminalu:
 cd vue-todo-client
 npm install
 npm run dev
