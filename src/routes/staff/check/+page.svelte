<script>
  export let data; // Receive data from the parent component
  import { calculateTravelTime } from '../../../lib/travelTimeCalculator.js';
  let tripsC = data.tripsC || []; // Store the staff-specific trips (default to an empty array if not provided)
  let selectedTrip = {}; // Store the selected trip
  let tripsQ = {}; // Store queried seat information
  let { stations } = data;

  console.log(tripsC)
  function formatDateTime(dateTime, onlyDate = false) {
    const date = new Date(dateTime);
    const formattedDate = date.toLocaleDateString('th-TH', {
      day: '2-digit',
      month: '2-digit',
      year: 'numeric',
    });
    
    if (onlyDate) {
      return formattedDate;
    }
    
    const formattedTime = date.toLocaleTimeString('th-TH', {
      hour: '2-digit',
      minute: '2-digit',
    }).replace(':', '.');

    return `${formattedTime} น. (${formattedDate})`;
  }

  // Function to handle trip selection
  async function handleTripSelection(trip) {
    selectedTrip = trip; // Store the selected trip
    console.log("Selected Trip:", selectedTrip); // Log selected trip details

    try {
      const response = await fetch("/staff/check", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ tripId: selectedTrip.trip_id }),
      });

      const result = await response.json();
      if (result.success) {
        tripsQ = result.tripsQ; // Store queried seat information
        console.log("TripsQ after fetch:", tripsQ); // Log tripsQ

        // Optional: Create query parameters from the selected trip data
        const query = new URLSearchParams({
          trip_name: selectedTrip.trip_name,
          trip_id: selectedTrip.trip_id,
          start: selectedTrip.start,
          start_id: selectedTrip.start_id, 
          end: selectedTrip.end,
          end_id: selectedTrip.end_id, 
          price: selectedTrip.price,
          from_datetime: formatDateTime(trip.from_datetime),
          arrivalTime: formatDateTime(
              calculateTravelTime(
                stations,
                trip.start, 
                trip.end, 
                trip.from_datetime
              )
            ),
          class: selectedTrip.class
        }).toString();

        // Redirect to /check/check2 with query parameters (if needed)
        window.location.href = `/staff/check/check2?${query}`;
      } else {
        console.error("Failed to fetch trip details from the backend.");
      }
    } catch (error) {
      console.error("Error while fetching trip details:", error);
    }
  }
</script>

<div class="min-h-screen bg-white sm:grid-cols-1">
  <h1 class="text-3xl font-bold mb-8 text-center text-[#102C57] underline py-8">
    ตรวจตั๋วโดยสาร
  </h1>
  <p class="text-lg text-[#102C57] sm:ml-32">
    สวัสดี! คุณ {tripsC[0].firstname} {tripsC[0].lastname} นี่คือเที่ยวโดยสารที่คุณต้องตรวจ
  </p>

  <div class="container mx-auto grid grid-cols-1 sm:grid-cols-2 gap-8">
    <!-- Display tripsC -->
    <div class="overflow-auto h-96">
      <h2 class="text-xl font-bold text-[#102C57] mb-4">
        เที่ยวโดยสารของเจ้าหน้าที่
      </h2>
      {#each tripsC as trip}
        <div class="bg-gray-200 sm:my-2 sm:p-2 my-4 p-4 rounded-lg flex sm:flex-col items-start sm:items-left justify-between">
          <div>
            <p class="text-base font-semibold">{formatDateTime(trip.from_datetime)} - {formatDateTime(
              calculateTravelTime(
                stations,
                trip.start, 
                trip.end, 
                trip.from_datetime
              )
            )}</p>
            <p class="text-sm">{trip.trip_id} {trip.start} - {trip.end}</p>
            <p class="text-sm">ชั้นโดยสารที่ {trip.class} ราคา {trip.price} บาท จำนวนที่นั่งเหลือ {trip.available_seats} ที่</p>

          </div>
          <button
            on:click={() => handleTripSelection(trip)}
            class="bg-[#102C57] text-white px-4 py-1 rounded-md mt-2 whitespace-nowrap text-sm"
          >
            ตรวจตั๋วโดยสาร
          </button>
        </div>
      {/each}
    </div>

    <div class="col-span-1 bg-white p-4">
      <h2 class="text-xl font-bold text-[#102C57] mb-4">
        เลือกเที่ยวโดยสารที่คุณกำลังจะตรวจ
      </h2>
      <p class="text-lg text-[#102C57] mb-4">
        และสแกน QR code จากตั๋วผู้โดยสาร อย่าลืม! เช็คผู้โดยสารให้ถูกต้อง
      </p>
    </div>
  </div>
</div>
