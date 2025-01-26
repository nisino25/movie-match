<template>
  <div id="app-container">

    <template v-if="currentPage == 0">
      <div class="w-full flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4">
        <h1 class="text-2xl font-bold mb-6">Choose Your Role</h1>
        <div class="flex flex-col space-y-4">
          <button
            class="px-4 py-2 bg-blue-500 text-white font-semibold rounded hover:bg-blue-600 text-xl"
            @click="selectRole('host')"
          >
            Be a Host
          </button>
          <button
            class="px-4 py-2 bg-green-500 text-white font-semibold rounded hover:bg-green-600 text-xl"
            @click="selectRole('guest')"
          >
            Be a Guest
          </button>
          <button
            class="px-4 py-2 bg-gray-500 text-white font-semibold rounded hover:bg-gray-600 text-xl"
            @click="selectRole('viewer')"
          >
            Be a Viewer
          </button>
        </div>

        <div v-if="role === 'host'" class="mt-6 p-4 bg-white shadow rounded">
          <h2 class="text-lg font-bold mb-4">Create Host Room</h2>
          <div class="space-y-2">
            <input
              type="text"
              v-model="username"
              placeholder="Enter your name"
              class="w-full px-3 py-2 border border-gray-300 rounded focus:outline-none focus:ring focus:ring-blue-500"
            />
            <button
              v-if="!isLoading"
              class="px-4 py-2 bg-blue-500 text-white font-semibold rounded hover:bg-blue-600 text-xl"
              @click="createHostRoom"
            >
              Generate Room Number
            </button>
          </div>
          <p v-if="isLoading">Loading...</p>
          <p v-if="errorMessage" class="mt-4 text-red-500">
            {{ errorMessage }}
          </p>
        </div>
        <div v-if="role === 'guest' && existingRoomNumbers.length > 0" class="mt-6 p-4 bg-white shadow rounded">
          <h2 class="text-lg font-bold mb-4">Enter Room number</h2>
          <div class="space-y-2">
            <input
              type="text"
              v-model="username"
              placeholder="Enter your name"
              class="w-full px-3 py-2 border border-gray-300 rounded focus:outline-none focus:ring focus:ring-blue-500"
            />
            <input
              type="number"
              v-model="roomNumber"
              placeholder="Enter room number"
              class="w-full px-3 py-2 border border-gray-300 rounded focus:outline-none focus:ring focus:ring-blue-500"
            />
            <button
              v-if="!isLoading"
              class="px-4 py-2 bg-blue-500 text-white font-semibold rounded hover:bg-blue-600 text-xl"
              @click="joinRoom"
            >
              Join Room
            </button>
          </div>
          <p v-if="isLoading">Loading...</p>
          <p v-if="errorMessage" class="mt-4 text-red-500">
            {{ errorMessage }}
          </p>
        </div>
        <div v-if="role === 'viewer' && existingRoomNumbers.length > 0" class="mt-6 p-4 bg-white shadow rounded">
          <h2 class="text-lg font-bold mb-4">Check Room</h2>
          <div class="space-y-2">
            <input
              type="text"
              v-model="roomNumber"
              placeholder="Enter room number"
              class="w-full px-3 py-2 border border-gray-300 rounded focus:outline-none focus:ring focus:ring-blue-500"
            />
            <button
              v-if="!isLoading"
              class="px-4 py-2 bg-blue-500 text-white font-semibold rounded hover:bg-blue-600 text-xl"
              @click="checkRoom"
            >
              Check Room
            </button>
          </div>
          <p v-if="isLoading">Loading...</p>
          <p v-if="errorMessage" class="mt-4 text-red-500">
            {{ errorMessage }}
          </p>
        </div>
      </div>
    </template>

    <template v-if="currentPage == 1">
      <div class="absolute top-0 left-0 w-full flex items-center justify-between bg-gray-800 text-white p-4 z-50">
        <!-- Profile Section -->
        <div class="flex items-center">
          <div class="w-10 h-10 bg-gray-500 rounded-full flex-shrink-0"></div>
          <span class="ml-2 text-lg font-semibold">{{username}}</span>
        </div>

        <!-- Picked Movies Section -->
        <div class="text-center" v-if="role == 'host'" >
          <span class="text-lg font-semibold">
            <span class="text-green-400">{{hostMoviesList.length}}</span> / <span>{{ maxMovieCount }}</span>
          </span>
        </div>

        <!-- Picked Movies Section -->
        <div class="text-center" v-if="role == 'guest'">
          <span class="text-lg font-semibold">
            <span class="text-green-400">{{guestMoviesList.length}}</span> / <span>{{ hostMoviesList.length }} from Host</span>
          </span>
        </div>

        <!-- Room Number Section -->
        <div class="flex items-center text-center">
          <span class="text-lg font-semibold bg-blue-500 px-3 py-1 rounded" @click="getSharedLink()">
            #{{ roomNumber || "00000" }}
            <small v-if="hostName"><br>{{ hostName }}</small>
          </span>
        </div>
      </div>

        <div class="tile-container">
          <template v-for="(tile, index) in tiles" :key="tile.id">
              <div 
                  v-if="index < currentTileIndex + 10"
                  :class="['tile', { 'swiped': isSwiped }]"
                  :style="{
                      width: tile.id === this.flyAwayTile ? '0px' : '',
                      height: tile.id === this.flyAwayTile ? '0px' : '',
                      transform: getTransform(tile, index),
                      zIndex: tiles.length - index,
                      transition: index === currentTileIndex && tile.id !== this.flyAwayTile && isSwiping ? 'unset' : '.4s ease-in-out',
                      display: index < currentTileIndex ? 'none' : ''
                  }"
                  @mousedown="startSwipe"
                  @mousemove="moveSwipe"
                  @mouseup="endSwipe"
                  @mouseleave="endSwipe"
                  @touchstart="startSwipe"
                  @touchmove="moveSwipe"
                  @touchend="endSwipe"
              >
                  <div class="img-container">
                      <img :src="'https://image.tmdb.org/t/p/w500' + tile.poster_path" :alt="tile.original_title" />
                  </div>
                  <div>
                      <!-- Display genres as badges -->
                      <div class="mt-1 overflow-hidden text-xs">
                          <span
                              v-for="(genreId, genreIndex) in tile.genre_ids"
                              :key="genreIndex"
                              class="bg-blue-500 text-white rounded-full px-3 py-1 text-sm font-medium inline mr-2">
                              {{ findGenreById(genreId).name }}
                          </span>
                      </div>
      
                      <span class="movie-title" v-if="tile.original_title" @click="getDetail(tile)"><small>{{index+1}} ({{ tile.release_date.slice(0, 4) }}) </small>{{ tile.original_title }}</span>
                  </div>
              </div>
          </template>

        </div>
    </template>

    <template v-if="currentPage == 10">
      <div v-if="matchedIds.length > 0" class="text-white">
        <h3>{{ hostName }} and {{ guestName }}'s matched {{ this.matchedIds.length }} movies</h3>
        <div class="grid grid-cols-2 gap-[20px] p-3 text-white">
          <template v-for="(tempTile,index) in matchedTiles" :key="index">
            <div class="tempTile" >
              <img :src="'https://image.tmdb.org/t/p/w500' + tempTile.poster_path" :alt="tempTile.original_title" />
              <hr>
              <span class="text-white text-sm" v-if="tempTile.original_title" @click="getDetail(tempTile)"><small>{{ tempTile.release_date.slice(0, 4) }}. </small>{{ tempTile.original_title }}</span>
            </div>
          </template>
        </div>
      </div>
      <div v-else>
        <h3>{{ hostName }} and {{ guestName }}'s dont have matched movie...</h3>
        {{ matchedIds }}
      </div>
      <hr>
      <div class="grid grid-cols-2 gap-[20px] p-3 text-white">
        <div>
          <template v-for="(tempTile,index) in hostMoviesTiles" :key="index">
            <div class="tempTile mb-5">
              <img :src="'https://image.tmdb.org/t/p/w500' + tempTile.backdrop_path" :alt="tempTile.original_title" />
              <span class="text-white text-xs" v-if="tempTile.original_title" @click="getDetail(tempTile)"><small>{{ index+1 }}. </small>{{ tempTile.original_title }}</span>
            </div>
          </template>
        </div>
        <div>
          <template v-for="(tempTile,index) in guestMoviesList" :key="index">
            <div class="tempTile mb-5">
              <img :src="'https://image.tmdb.org/t/p/w500' + tempTile.backdrop_path" :alt="tempTile.original_title" />
              <span class="text-white text-xs" v-if="tempTile.original_title" @click="getDetail(tempTile)"><small>{{ index+1 }}. </small>{{ tempTile.original_title }}</span>
            </div>
          </template>
        </div>
      </div>

      <button class="mt-5" @click="backToSelecting()">Back</button>
    </template>

    <template v-if="currentPage == 999">
      <div class="grid grid-cols-2 gap-[25px] p-5">
        <template v-for="(tempTile,index) in tempTiles" :key="index">
          <div class="tempTile" v-if="index >= (pagination - 1) * 500 && index < pagination * 500" @click="getDetail(tempTile)">
            <img :src="'https://image.tmdb.org/t/p/w500' + tempTile.poster_path" :alt="tempTile.original_title" />
            <hr>
            <span class="movie-title" v-if="tempTile.original_title"><small>{{index+1}} ({{ tempTile.release_date.slice(0, 4) }}) </small>{{ tempTile.original_title }}</span>
          </div>
        </template>
        <div class="button-container">
          <button @click="pagination--">-</button>
          <strong>{{pagination}}</strong>
          <button @click="pagination++">+</button>
        </div>
      </div>
    </template>
  </div>

</template>

<script>
import { moviesInternational } from '../public/const/moviesInternational';
import { moviesJapan } from '../public/const/moviesJapan';
export default {
  data() {
  return {
      moviesInternational,
      moviesJapan,
      currentPage: 0,
      tiles: null,
      startX: 0,
      startY: 0,
      moveX: 0,
      moveY: 0,

      isSwiping: false,
      isSwiped: false,
      flyAwayTile: null,

      bonusPoints: 0,
      currentTileIndex: 0,
      timer: 0,
      defaultTime: 4,
      timerInterval: null,
      hasGameFinished: false,
      genres : [
          {
          "id": 28,
          "name": "Action"
          },
          {
          "id": 12,
          "name": "Adventure"
          },
          {
          "id": 16,
          "name": "Animation"
          },
          {
          "id": 35,
          "name": "Comedy"
          },
          {
          "id": 80,
          "name": "Crime"
          },
          {
          "id": 99,
          "name": "Documentary"
          },
          {
          "id": 18,
          "name": "Drama"
          },
          {
          "id": 10751,
          "name": "Family"
          },
          {
          "id": 14,
          "name": "Fantasy"
          },
          {
          "id": 36,
          "name": "History"
          },
          {
          "id": 27,
          "name": "Horror"
          },
          {
          "id": 10402,
          "name": "Music"
          },
          {
          "id": 9648,
          "name": "Mystery"
          },
          {
          "id": 10749,
          "name": "Romance"
          },
          {
          "id": 878,
          "name": "SF"
          },
          {
          "id": 10770,
          "name": "TV Movie"
          },
          {
          "id": 53,
          "name": "Thriller"
          },
          {
          "id": 10752,
          "name": "War"
          },
          {
          "id": 37,
          "name": "Western"
          }
      ],

      uniqueId: null,
      currentUserRow: null,
      currentUsername: null,

      currentTotalPoints: null,

      computedHref: null,
      
      hasPointsReached: null,

      developingMode: true,
      role: null,
      username: "",
      roomNumber: null,
      errorMessage: "",
      baseUrl: "https://script.google.com/macros/s/AKfycbwDnmUz6jgLDWZ5ZQFhUBOrW-iQM6w8MvA51CBeO2cCWxat-lIhcjWgXmfnm4_O1jPAvg/exec",
      isLoading: false,
      existingRoomNumbers: [], 

      maxMovieCount: 25,
      hostMoviesList: [],

      hostName: null,
      guestMoviesList: [],

      tempTiles: [],
      pagination: 1,

      guestName: null,
      matchedIds: [],
      matchedTiles: [],
      hostMoviesTiles: [],
      guestMoviesTiles: [],
  };
},
watch: {
  currentPage(newPage) {
      const appElement = document.getElementById('app');
      if (newPage === 0) {
          // Apply styles if currentPage is 0
          appElement.style.display = 'block';
          appElement.style.marginTop = '20px';
      } else {
          // Remove styles if currentPage is not 0
          appElement.style.display = '';
          appElement.style.marginTop = '';
      }
  },
},
computed: {
},
methods: {
  async startGame(){  
    // await this.incrementSorting()
    this.hasGameFinished = false;
    // this.currentPage = 1
    this.tiles = this.generateRandomTiles()
    this.currentTileIndex = 0
  },
  generateRandomTiles() {

    let combinedArr = [...this.moviesJapan, ...this.moviesInternational];
    // console.log(combinedArr)


    

      // Generate random tiles
      const tiles = this.shuffleArray(combinedArr);

      return tiles;
  },
  shuffleArray(array) {
    
    for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]]; // Swap elements
    }
    return array;
  },
  findGenreById(id) {
    return this.genres.find(genre => genre.id === id);
},
  startSwipe(event) {
      if (event.type.startsWith('mouse') && event.button !== 0) return;
      this.isSwiping = true;
      const point = event.touches ? event.touches[0] : event;
      this.startX = point.clientX;
      this.startY = point.clientY;
  },
  moveSwipe(event) {
      if (!this.isSwiping) return;
      event.preventDefault();
      const point = event.touches ? event.touches[0] : event;
      this.moveX = point.clientX - this.startX;
      this.moveY = point.clientY - this.startY;
  },
  startTimer() {
      this.timer = this.tiles.length * 1.5;
      clearInterval(this.timerInterval);
      this.timerInterval = setInterval(() => {
          this.timer = Math.max(0, this.timer - 0.1);
          if (this.timer === 0) {
              clearInterval(this.timerInterval);
          }
      }, 100);
  },
  async endSwipe() {
      if (!this.isSwiping && this.timer > 0) return;

      
      // const threshold = 100;
      const minMove = 50;
      const minYmove = 150;
      // const currentCategory = this.tiles[this.currentTileIndex].category;
      // let success = false;

      if (Math.abs(this.moveY) > minYmove) return this.failSwipe(Math.abs(this.moveY))

      if (Math.abs(this.moveX) > minMove) {
        // success = true
        this.flyAwayTile = this.tiles[this.currentTileIndex];
        if(this.moveX > minMove){
            if(this.role == 'host') this.addMovie()
            if(this.role == 'guest') this.compareList()
            document.body.style.backgroundColor = 'forestGreen';  // Change background color to forest green
            document.body.style.backgroundImage = 'none'; // Remove gradient image
            if(this.flyAwayTile) this.flyAwayTile.isPicked = true;
        }else{
            document.body.style.backgroundColor = 'red';  // Change background color to red
            document.body.style.backgroundImage = 'none'; // Remove gradient image
            if(this.flyAwayTile) this.flyAwayTile.isPicked = false;
        }

                
        this.isSwiped = true;
        setTimeout(() => {
        this.currentTileIndex++;
        this.flyAwayTile = null;
        this.isSwiped = false;
            
        }, 400);

        setTimeout(() => {
          document.body.style.backgroundColor = 'transparent';  // Reset the background color to the default (transparent or inherited)
          document.body.style.backgroundImage = 'rgb(20, 30, 48)';  // Apply the gradient
        }, 300);  // Transition will occur smoothly
      }

      this.isSwiping = false;
      this.moveX = 0;
      this.moveY = 0;
  },
  failSwipe(num){
    console.log(num)
    this.isSwiping = false;
    this.moveX = 0;
    this.moveY = 0;
  },
  getTransform(tile, index) {
      const adjustedIndex = index - this.currentTileIndex;
      if (adjustedIndex === 0 && this.isSwiping) {
          return `translate(${this.moveX}px, ${this.moveY}px) rotate(${this.moveX * 0.1}deg)`;
      }
      if (tile.id === this.flyAwayTile?.id) {
        // let direction
        if(this.flyAwayTile.isPicked){
            return `translate(500px, 50px) rotate(45deg)`;
        }else{
            return `translate(-500px, 50px) rotate(-45deg)`;
        }
      }
      const rotation = adjustedIndex * 5
      return `scale(${1 - adjustedIndex * 0.05}) translateY(${-adjustedIndex * 0}px) rotate(${rotation}deg)`;
  },

  async loadRoomNumbers() {
    console.log('loading rooms')
    try {
      const response = await fetch(
        `${this.baseUrl}?action=getAllRooms`
      );
      const result = await response.json();

      if (result.success && Array.isArray(result.rooms)) {
        
        this.existingRoomNumbers = result.rooms;
      } else {
        this.errorMessage = "Failed to load room numbers.";
      }
    } catch (error) {
      console.error(error);
      this.errorMessage = "An error occurred while loading room numbers.";
    }
  },

  selectRole(role) {
    this.role = role;
  },
  async createHostRoom() {
    

      if (!this.username) {
          this.errorMessage = "Please enter a username!";
          return;
      }

      this.isLoading = true

      this.errorMessage = "";
      this.roomNumber = null;

      // Generate a list of all possible room numbers (10000 to 99999)
      const allRoomNumbers = Array.from({ length: 90000 }, (_, i) => (i + 10000).toString());

      // Filter out the ones that already exist
      const availableRoomNumbers = allRoomNumbers.filter(
          (roomNumber) => !this.existingRoomNumbers.includes(roomNumber)
      );

      // Check if there are any available room numbers
      if (availableRoomNumbers.length === 0) {
          this.errorMessage = "No available room numbers. Please try again later.";
          return;
      }

      // Randomly select one from the available room numbers
      const randomIndex = Math.floor(Math.random() * availableRoomNumbers.length);
      const selectedRoomNumber = availableRoomNumbers[randomIndex];

      // Add room number and username to the sheet
      const success = await this.addRoomToSheet(selectedRoomNumber, this.username);
      if (success) {
          this.roomNumber = selectedRoomNumber;
          this.currentPage = 1;
          this.isLoading = false
      }
  },
  async addRoomToSheet(roomNumber, username) {
    try {
      const response = await fetch(
        `${this.baseUrl}?action=addRoom&roomId=${roomNumber}&hostName=${username}`
      );
      const result = await response.json();
      if (result.success) {
        // Update local list of room numbers
        this.existingRoomNumbers.push(roomNumber.toString());
        return true;
      } else {
        this.errorMessage = "Failed to create room. Please try again.";
        return false;
      }
    } catch (error) {
      console.error(error);
      this.errorMessage = "An error occurred while creating the room.";
      return false;
    }
  },
  addMovie(){
    this.hostMoviesList.push(this.flyAwayTile.id)
    if (this.hostMoviesList.length % 3 === 0 || this.hostMoviesList.length === this.maxMovieCount) {
        this.sendMoviesToSpreadsheet();
    }
  },
  async sendMoviesToSpreadsheet() {
    try {
        // Send the hostMoviesList as a JSON string
        if(this.hostMoviesList.length == this.maxMovieCount) alert('Done!')
        const moviesJSON = JSON.stringify(this.hostMoviesList);

        // Make a request to the Google Apps Script to update the spreadsheet
        const response = await fetch(
            `${this.baseUrl}?action=updateMovies&roomNumber=${this.roomNumber}&movies=${encodeURIComponent(moviesJSON)}`
        );
        
        

        const result = await response.json();
        if (result.success) {
            console.log("Movies successfully sent to the spreadsheet!");
        } else {
            console.error("Failed to send movies to the spreadsheet:", result.message);
        }
    } catch (error) {
        console.error("An error occurred while sending movies to the spreadsheet:", error);
    }
  },

  async joinRoom() {
    

    if (!this.username) {
        this.errorMessage = "Please enter a username!";
        return;
    }

    if (!this.roomNumber) {
        this.errorMessage = "Please enter a roomnumber!";
        return;
    }

    this.isLoading = true
    this.errorMessage = "";

    try {
        // Check if the room exists and is not occupied
        const room = this.existingRoomNumbers.find(
            (room) => room.roomId === this.roomNumber
        );

        if (!room) {
            this.errorMessage = "Room does not exist. Please check the room number.";
            this.isLoading = false;
            return;
        }

        if (!room.isColEEmpty) {
            this.errorMessage = "The room is already occupied. Please try another room.";
            this.isLoading = false;
            return;
        }

        // Join the room by marking it as occupied locally and updating the sheet
        const success = await this.addUserToRoom(this.roomNumber, this.username);
        if (success) {
            room.isColEEmpty = false; // Mark room as occupied locally
            this.currentPage = 1; // Navigate to the next page
            this.hostName = room.hostName
            this.hostMoviesList = room.hostMoviesList ? JSON.parse(room.hostMoviesList) : []; 
            console.log(this.hostMoviesList)
            this.hostAvatar = room.hostAvatar
            
        
        } else {
            this.errorMessage = "Failed to join the room. Please try again.";
        }
    } catch (error) {
        console.error(error);
        this.errorMessage = "An error occurred while joining the room.";
    } finally {
        this.isLoading = false;
    }
  },
  async addUserToRoom(roomNumber, username) {
      try {
          const response = await fetch(
              `${this.baseUrl}?action=addUserToRoom&roomId=${roomNumber}&guestName=${username}`
          );
          const result = await response.json();
          return result.success;
      } catch (error) {
          console.error(error);
          this.errorMessage = "An error occurred while adding the user to the room.";
          return false;
      }
  },
  compareList(){
    this.guestMoviesList.push(this.flyAwayTile.id)

    // Check for matches between guestMoviesList and hostMoviesList
    const matchingId = this.hostMoviesList.find((id) => this.guestMoviesList.includes(id));
    // const matchingId = this.guestMoviesList[0]

    if (matchingId) {
        // Remove the matching ID from hostMoviesList
        const index = this.hostMoviesList.indexOf(matchingId);
        if (index !== -1) {
            this.hostMoviesList.splice(index, 1); // Remove 1 item at the found index
        }
        // Combine movies from both arrays
        const combinedArr = [...this.moviesJapan, ...this.moviesInternational];

        // Find the movie by ID in the combined array
        const matchingMovie = combinedArr.find((movie) => movie.id === matchingId);

        if (matchingMovie) {
            // Alert the movie title
            alert(`Match found! Movie: ${matchingMovie.title}`);
        }
    }

    if(this.guestMoviesList.length % 3 === 0 || matchingId) this.sendGuestMovies();

  },

  async sendGuestMovies() {
    try {
        const moviesJSON = JSON.stringify(this.guestMoviesList);

        // Make a request to the Google Apps Script to update the spreadsheet
        const response = await fetch(
            `${this.baseUrl}?action=updateGuestMovies&roomNumber=${this.roomNumber}&movies=${encodeURIComponent(moviesJSON)}`
        );
        
        

        const result = await response.json();
        if (result.success) {
            console.log("Movies successfully sent to the spreadsheet!");
        } else {
            console.error("Failed to send movies to the spreadsheet:", result.message);
        }
    } catch (error) {
        console.error("An error occurred while sending movies to the spreadsheet:", error);
    }
  },

  getSharedLink() {
      // Step 1: Get the current domain
      const currentDomain = window.location.origin;

      // Step 2: Generate the shareable link
      const shareableLink = `${currentDomain}?roomNumber=${this.roomNumber}`;
      console.log(shareableLink)

      // Step 3: Copy the link to the clipboard
      navigator.clipboard.writeText(shareableLink)
          .then(() => {
              alert("Link copied to clipboard!");
          })
          .catch((error) => {
              console.error("Failed to copy link:", error);
              alert("Failed to copy the link. Please try again.");
          });
  },
  checkLink() {
      // Step 1: Get the current URL search parameters
      const urlParams = new URLSearchParams(window.location.search);

      // Step 2: Check if the "roomNumber" parameter exists
      if (urlParams.has("roomNumber")) {
          // Step 3: Set the roomNumber and role
          this.roomNumber = Number(urlParams.get("roomNumber"));
          console.log(typeof this.roomNumber)
          this.role = "guest";
          console.log(`Room Number: ${this.roomNumber}, Role: ${this.role}`);
      } else {
          console.log("No room number found in the URL.");
      }
  },

  getDetail(tile){
    let description = `
      ${tile.release_date}. ${tile.original_title} \n
      Category: ${this.displayGenres(tile)}\n
      ${tile.overview}
    `
    alert(description)
  },
  displayGenres(tile) {
    // Map genre_ids to their names
    const genreNames = tile.genre_ids
      .map((genreId) => this.findGenreById(genreId).name)
      .join(", ");

    console.log("Genres:", genreNames); // Log genres
    return genreNames; // Return for display
  },

  async checkRoom() {
    console.log('checking room');

    if (!this.roomNumber) {
        this.errorMessage = "Please enter a roomnumber!";
        return;
    }

    this.isLoading = true
    this.errorMessage = "";

    // Check if the room exists and is not occupied
    const room = this.existingRoomNumbers.find(
        (room) => room.roomId === this.roomNumber
    );

    if (!room) {
        this.errorMessage = "Room does not exist. Please check the room number.";
        this.isLoading = false;
        return;
    }


    this.currentPage = 10; // Navigate to the next page
    this.hostName = room.hostName
    this.hostMoviesList = room.hostMoviesList ? JSON.parse(room.hostMoviesList) : []; 

    this.guestName = room.guestName || null;
    this.guestMoviesList = room.guestMoviesList ? JSON.parse(room.guestMoviesList) : []; 
    

    this.matchedIds = Array.from(
      new Set(this.hostMoviesList.filter((hostMovieId) => 
        this.guestMoviesList.includes(hostMovieId)
      ))
    );

    

    this.matchedTiles = this.tiles.filter(tile => this.matchedIds.includes(tile.id));
    this.hostMoviesTiles = this.tiles.filter(tile => this.hostMoviesList.includes(tile.id));
    this.guestMoviesList = this.tiles.filter(tile => this.guestMoviesList.includes(tile.id));


    
    document.body.style.overflow = 'auto !important';
    document.body.style.padding = '1em';
    
    
    document.getElementById('app-container').style.display = 'block';

    this.isLoading = false;
  },

  backToSelecting(){
    document.body.style.overflow = 'hidden !important';
    document.body.style.padding = '0';
    document.getElementById('app-container').style.display = 'flex';

    this.role = null;
    this.currentPage = 0;
    this.roomNumber = null
  },






},
  
  async mounted() {
    console.clear()
    
    this.loadRoomNumbers();
    this.checkLink();
    this.startGame();

    this.role = 'viewer'
    // this.role = 'guest'
    this.roomNumber = Number(45102)
    // this.roomNumber = '76113'

    // this.currentPage = 999;
    // this.tempTiles = [...this.moviesJapan, ...this.moviesInternational];
    // this.pagination = 4;
  },
};
</script>

<style>
  body.device-mobile-optimized>*{
      max-width: unset !important;
  }
  body, html {
      margin: 0;
      padding: 0;
      height: 100%;
      transition: all .25s ease-in-out;

      overflow: hidden !important;
      
      font-size: unset;
      background: rgb(20, 30, 48);
  }

  #app{
    margin-top: unset !important;
  }

  #app-container {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100dvh;
      transition: background-color 0.3s ease;
  }

  button{
      background-color: #3498db; /* Primary color */
      color: white; /* Text color */
      border: none; /* Remove default border */
      border-radius: 4px; /* Rounded corners */
      padding: 5px 20px; /* Padding */
      font-size: 12px; /* Font size */
      cursor: pointer; /* Pointer cursor on hover */
      transition: background-color 0.3s, transform 0.2s; /* Smooth transitions */
  }

  /* ------------------------------------------------------------------- */

  .tile-container {
      position: relative;
      width: 70vw;
      aspect-ratio: 5/9;
      /* height: 100px; */

      /* margin-top: 125px; */
  }
  .tile {
    width: 100%;
    height: 100%;
    padding: 1em;
    position: absolute;
    top: 0;
    left: 0;
    border-radius: 10px;
    text-align: center;
    font-size: 18px;
    color: #333;  /* Dark text for contrast */
    background-color: white;  /* Light card background */
    user-select: none;
    border: 2px solid transparent;
    transition: all 0.3s ease-in-out;
    overflow: hidden;
    box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2);
    cursor: pointer;
    transform: scale(1);
}

.tile:hover {
    transform: scale(1.05);
    box-shadow: 0 6px 30px rgba(0, 0, 0, 0.3);
}

.tile .img-container{
    display: block;
    width: 100%;
}

.tile img {
    
    display: block;
    margin: auto;
    height: auto;
    object-fit: cover;
    border-radius: 10px;
    margin-bottom: 10px;
}

.movie-title {
    text-align: left;
    font-size: 1.5rem;
    display: -webkit-box;
    -webkit-line-clamp: 2; /* Limit to 2 lines */
    -webkit-box-orient: vertical;
    line-height: 1.25;
    overflow: hidden; /* Hide any text that overflows */
    text-overflow: ellipsis; /* Add the ellipsis (...) */
    margin: auto;
    margin-top: 10px;
    color: #333;
}


.tile p {
    font-size: 16px;
    color: #666;
    margin: 10px 0;
}

.tile .details {
    font-size: 14px;
    color: #999;
    margin-top: 10px;
    opacity: 0.8;
}

.tile .rating {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 5px;
    font-size: 18px;
    margin-top: 5px;
}

.tile .rating i {
    font-size: 22px;
    color: #f39c12;
}

.tile .rating span {
    font-weight: bold;
}

  
  .swiped {
      transition: transform 0.3s ease;
  }

  .top-block{

      position: absolute;
      top: 0px;
      left: 50%;
      transform: translate(-50%, -275px);

      z-index: 100;
  }
  
  .detail{
      user-select: none;
      pointer-events: none;
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      justify-content: space-between;
      align-items: center;

      text-align: center;

      width: 100%;
      min-width: 300px;

      font-weight: bold;
      font-size: 1.5em;
  }

  .result{
      font-weight: bold;
      text-align: center;
      font-size: 1.5em;
  }

  .result mark{
      background-color: unset;
      font-size: 1.5em;
      color: crimson;
      margin-left: 5px;

  }

  .result button{
      margin-top: 20px;
  }

  .timer {
      font-size: 24px;
      font-weight: bold;
      color: black;
  }
</style>