# Doctor-Appointment-booking-system
/**
 * DOCTOR APPOINTMENT BOOKING SYSTEM
 * Complete logic contained in a single executable JavaScript file.
 */

// --- DATA LAYER (Memory Store) ---
const database = {
    doctors: [
        { id: 1, name: "Dr. Alice Smith", specialty: "Cardiologist" },
        { id: 2, name: "Dr. Bob Jones", specialty: "Dermatologist" }
    ],
    appointments: [] // Holds finalized bookings
};

// --- CORE SYSTEM ENGINE ---
const BookingSystem = {
    
    // 1. Fetch and display all available doctors
    getDoctors() {
        return database.doctors;
    },

    // 2. Core validation to check if a specific doctor is busy at a chosen time
    isSlotConflict(doctorId, date, time) {
        return database.appointments.some(appt => 
            appt.doctorId === doctorId && 
            appt.date === date && 
            appt.time === time
        );
    },

    // 3. Create a unique, verifiable reference ID for bookings
    generateTrackingId() {
        return 'APT-' + Math.random().toString(36).substr(2, 9).toUpperCase();
    },

    // 4. Main booking pipeline with validation checks
    bookAppointment(patientName, doctorId, date, time) {
        // Validate doctor existence
        const doctorExists = database.doctors.find(doc => doc.id === doctorId);
        if (!doctorExists) {
            return { success: false, message: `Error: Doctor ID ${doctorId} not found.` };
        }

        // Validate scheduling conflict
        if (this.isSlotConflict(doctorId, date, time)) {
            return { 
                success: false, 
                message: `Error: ${doctorExists.name} is already booked on ${date} at ${time}.` 
            };
        }

        // Generate booking object
        const newAppointment = {
            trackingId: this.generateTrackingId(),
            patientName: patientName,
            doctorId: doctorId,
            doctorName: doctorExists.name,
            specialty: doctorExists.specialty,
            date: date,
            time: time,
            timestamp: new Date().toISOString()
        };

        // Save record
        database.appointments.push(newAppointment);

        return { 
            success: true, 
            message: "Success: Appointment confirmed!", 
            data: newAppointment 
        };
    },

    // 5. Query active appointments by patient name
    getPatientSchedule(patientName) {
        return database.appointments.filter(appt => 
            appt.patientName.toLowerCase() === patientName.toLowerCase()
        );
    },

    // 6. Delete an appointment using its reference ID
    cancelAppointment(trackingId) {
        const index = database.appointments.findIndex(appt => appt.trackingId === trackingId);
        if (index === -1) {
            return { success: false, message: "Error: Appointment tracking ID not found." };
        }
        
        database.appointments.splice(index, 1);
        return { success: true, message: `Success: Appointment ${trackingId} cancelled.` };
    }
};

// ============================================================================
// SIMULATION & DEMONSTRATION RUN
// ============================================================================
console.log("=== 1. AVAILABLE CLINIC DOCTORS ===");
console.log(BookingSystem.getDoctors());
console.log("\n");

console.log("=== 2. ATTEMPTING APPOINTMENT BOOKINGS ===");
// Successful Booking 1
const booking1 = BookingSystem.bookAppointment("John Doe", 1, "2026-08-15", "10:00 AM");
console.log(booking1.message, booking1.data ? `ID: ${booking1.data.trackingId}` : "");

// Successful Booking 2
const booking2 = BookingSystem.bookAppointment("Jane Miller", 1, "2026-08-15", "11:00 AM");
console.log(booking2.message, booking2.data ? `ID: ${booking2.data.trackingId}` : "");

// Failure Booking: Double-booking prevention check
const booking3 = BookingSystem.bookAppointment("Alex Wong", 1, "2026-08-15", "10:00 AM");
console.log(booking3.message); // Should fail due to time conflict
console.log("\n");

console.log("=== 3. VIEWING ACTIVE BOOKINGS BY PATIENT ===");
console.log("John Doe's List:", BookingSystem.getPatientSchedule("John Doe"));
console.log("\n");

console.log("=== 4. SYSTEM APPOINTMENT CANCELLATION ===");
if (booking1.success) {
    const cancelAction = BookingSystem.cancelAppointment(booking1.data.trackingId);
    console.log(cancelAction.message);
}
console.log("\n");

console.log("=== 5. RETRYING PREVIOUSLY CONFLICTED SLOT ===");
// This should pass now because John Doe's 10:00 AM slot was cancelled
const booking4 = BookingSystem.bookAppointment("Alex Wong", 1, "2026-08-15", "10:00 AM");
console.log(booking4.message, booking4.data ? `ID: ${booking4.data.trackingId}` : "");
