Ordering system code:

package HostelManagementSystem;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Scanner;

public class HospitalManagementSystemDriver {
	private static final String url = "jdbc:mysql://localhost:3306/hospital_management";
	private static final String username = "root";
	private static final String password = "root";
	//static Scanner scanner;
	//private static Scanner scanner;

	public static void main(String[] args) {
		// scanner = new Scanner(System.in);

		try {
			Class.forName("com.mysql.cj.jdbc.Driver");
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		 Scanner scanner = new Scanner(System.in);

		
		try {
			Connection connection = DriverManager.getConnection(url,username,password);
			Patient patient = new Patient(connection, scanner);
			Doctor doctor = new Doctor(connection);
			while(true) {
				System.out.println("HOSPITAL MANAGEMENT SYSTEM");
				System.out.println("1. Add Patient");
				System.out.println("2. View Patients");
				System.out.println("3. View Doctors");
				System.out.println("4. Book Appointment");
				System.out.println("5. Exit");
				System.out.println("Enter Your Choice");
				int choice = scanner.nextInt();
				
				switch(choice) {
				case 1:
					//add Patient
					patient.addPatient();
					System.out.println();
					break;
				case 2:
					//view Patient
					patient.viewPatients();
					System.out.println();
					break;
				case 3:
					//view Doctors
					doctor.viewDoctors();
					System.out.println();
					break;
				case 4:
					//Book Appointment
					bookAppointment(patient, doctor, connection, scanner);
					System.out.println();
					break;
					
				case 5:
					//exit
					System.out.println("Thank You for using Hospital Management System");
					return;
					default:
						System.out.println("Please Enter Valid Choice");
						break;
				}
			}
		}catch(SQLException e) {
			e.printStackTrace();
		}
		
		
	}
	public static void bookAppointment(Patient patient, Doctor doctor,Connection connection, Scanner scanner) {
		System.out.println("Enter Patient Id");
		int patientId = scanner.nextInt();
		System.out.println("Enter doctor Id");
		int doctorId = scanner.nextInt();
		System.out.println("Enter appointment date(YYYY-MM-DD): ");
		String appointmentDate = scanner.next();
		if(patient.getPatientById(patientId) && doctor.getDoctorById(doctorId)) {
			if(checkDoctorAvailability(doctorId, appointmentDate, connection)) {
				String appointmentQuery = "INSERT INTO appointments(patient_id, doctor_id, appointment_id) VALUES(?,?,?)";
				try {
					PreparedStatement preparedStatement = connection.prepareStatement(appointmentQuery);
					preparedStatement.setInt(1,patientId);	
					preparedStatement.setInt(2,doctorId);
					preparedStatement.setString(3,appointmentDate);
					int rowsAffected = preparedStatement.executeUpdate();
					if(rowsAffected>0) {
						System.out.println("Appointment Booked");
					}else {
						System.out.println("Failed to Book Appointment");
					}
				}catch(Exception e) {
					e.printStackTrace();
				}
				
			}else {
				System.out.println("Doctor Not Available This date");
			}
		}else {
			System.out.println("Either Doctor or Patient doesn't exits");
		}
		
	}
	public static boolean checkDoctorAvailability(int doctorId, String appointmentDate, Connection connection) {
		String query = "SELECT COUNT(*) FROM appointments WHERE doctor_id = ? AND appointment_date = ?";
		try {
			PreparedStatement preparedStatement = connection.prepareStatement(query);
			preparedStatement.setInt(1,doctorId);
			preparedStatement.setString(2, appointmentDate);
			ResultSet resultSet = preparedStatement.executeQuery();
			if(resultSet.next()) {
				int count = resultSet.getInt(1);
				if(count==0) {
					return true;
				}else {
					return false;
				}
			}
			
			
		}catch(Exception e) {
			e.printStackTrace();
		}
		return false;
	}

}
