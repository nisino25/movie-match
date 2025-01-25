<template>
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
          v-if="!roomNumber && !isLoading"
          class="px-4 py-2 bg-blue-500 text-white font-semibold rounded hover:bg-blue-600 text-xl"
          @click="createHostRoom"
        >
          Generate Room Number
        </button>
      </div>
      <p v-if="isLoading">Loading...</p>
      <p v-if="roomNumber" class="mt-4 text-green-500 font-bold">
        Room Created!<br>Room Number: {{ roomNumber }}
      </p>
      <p v-if="errorMessage" class="mt-4 text-red-500">
        {{ errorMessage }}
      </p>
    </div>
    <div v-if="role === 'guest'" class="mt-6 p-4 bg-white shadow rounded">
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
      <div class="text-center" v-if="role == 'host'">
        <span class="text-lg font-semibold">
          <span class="text-green-400">{{hostMoviesList.length}}</span> / <span>{{ maxMovieCount }}</span>
        </span>
      </div>

      <!-- Picked Movies Section -->
      <div class="text-center" v-if="role == 'guest'">
        <span class="text-lg font-semibold">
          <span class="text-green-400">{{guestMovieList.length}}</span> / <span>{{ hostMoviesList.length }} from Host</span>
        </span>
      </div>

      <!-- Room Number Section -->
      <div class="flex items-center text-center">
        <span class="text-lg font-semibold bg-blue-500 px-3 py-1 rounded">
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
                    <div class="mt-1">
                        <span
                            v-for="(genreId, genreIndex) in tile.genre_ids"
                            :key="genreIndex"
                            class="bg-blue-500 text-white rounded-full px-3 py-1 text-sm font-medium inline mr-2">
                            {{ findGenreById(genreId).name }}
                        </span>
                    </div>
    
                    <span class="movie-title" v-if="tile.original_title"><small>{{index+1}} ({{ tile.release_date.slice(0, 4) }}) </small>{{ tile.original_title }}</span>
                </div>
            </div>
        </template>

      </div>
  </template>

  <template v-if="currentPage == 2">
      <div class="result">
          <div>
              <small v-if="currentUsername">現在の合計<mark>{{currentTotalPoints}}</mark>点</small>
              <hr>
              <small style="font-weight: unset; display: block; text-align: right;">
                  スピードボーナス<mark>+{{bonusPoints}}</mark>点<br>
                  かんりょうボーナス<mark>+2</mark>点
              </small>
              <hr>
              <button @click="startGame">リスタート</button>
          </div>
      </div>
  </template>
</template>

<script>
import { movie_data } from '../public/const/movieData';
import { moviesInternational } from '../public/const/moviesInternational';
import { moviesJapan } from '../public/const/moviesJapan';
export default {
  data() {
  return {
        movie_data,
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
        baseUrl: "https://script.google.com/macros/s/AKfycbx9uUjKDxpopuCig5ukgaV36EKgZU2yYW6VuNY3QJNMT0S0Y72VLuYbrNTXuizOcAVprQ/exec",
        isLoading: false,
        existingRoomNumbers: [], 

        maxMovieCount: 20,
        hostMoviesList: [],

        hostName: null,
        guestMovieList: [],
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
    this.tiles = this.generateRandomTiles(15)
    this.currentTileIndex = 0
  },
  shuffleArray(array) {
        for (let i = array.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [array[i], array[j]] = [array[j], array[i]]; // Swap elements
        }
        return array;
    },
  removeAdultsAndShuffle(array) {
    // Filter out items where .adult is true
    const filteredArray = array.filter(item => !item.adult);
    // Shuffle the filtered array
    return this.shuffleArray(filteredArray);
},
  generateRandomTiles() {

    let combinedArr = [...this.moviesJapan, ...this.moviesInternational];
    // console.log(combinedArr)


    

      // Generate random tiles
      const tiles = this.removeAdultsAndShuffle(combinedArr)
      console.log(tiles)

      return tiles;
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
      // const currentCategory = this.tiles[this.currentTileIndex].category;
      // let success = false;

      if (Math.abs(this.moveY) > minMove) return this.failSwipe()

      if (Math.abs(this.moveX) > minMove) {
        // success = true
        this.flyAwayTile = this.tiles[this.currentTileIndex];
        if(this.moveX > minMove){
            if(this.role == 'host') this.addMovie()
            if(this.role == 'guest') this.compareList()
            console.log(this.flyAwayTile)
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

                if (this.currentTileIndex == this.tiles.length-1 && !this.hasGameFinished) {
                this.hasGameFinished = true
                    document.body.style.backgroundColor = 'yellow';
                    clearInterval(this.timerInterval);
                    this.bonusPoints = Math.floor(this.timer);

                
                    this.currentPage =2
                }

                setTimeout(() => {
            document.body.style.backgroundColor = 'transparent';  // Reset the background color to the default (transparent or inherited)
            document.body.style.backgroundImage = 'rgb(20, 30, 48)';  // Apply the gradient
        }, 300);  // Transition will occur smoothly
      }

      this.isSwiping = false;
      this.moveX = 0;
      this.moveY = 0;
  },
  failSwipe(){
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
        console.log('got the room data')
        console.log(this.existingRoomNumbers)
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
    if (this.hostMoviesList.length % 5 === 0 || this.hostMoviesList.length === this.maxMovieCount) {
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
    this.guestMovieList.push(this.flyAwayTile.id)

    // Check for matches between guestMovieList and hostMoviesList
    const matchingId = this.hostMoviesList.find((id) => this.guestMovieList.includes(id));
    // const matchingId = this.guestMovieList[0]

    if (matchingId) {
        // Combine movies from both arrays
        const combinedArr = [...this.moviesJapan, ...this.moviesInternational];

        // Find the movie by ID in the combined array
        const matchingMovie = combinedArr.find((movie) => movie.id === matchingId);

        if (matchingMovie) {
            // Alert the movie title
            alert(`Match found! Movie: ${matchingMovie.title}`);
        }
    }


  },




},
  
  async mounted() {
    console.clear()
    // console.log(this.movie_data)
      // this.getParams();
      // if(this.uniqueId) await this.findMe()
      // this.currentPage = 1
      this.loadRoomNumbers();
      if(this.developingMode){
        // this.role = 'guest';
        // this.role = 'host';
        // this.roomNumber = 76113;
        // this.username = 'sunshine';
        // this.currentPage = 1;
      }

      this.startGame()
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

  #app {
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

.tile .movie-title {
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