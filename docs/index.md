FTC OnBotJava Mecanum Wheel Programming Guide 

Step 1: Physical Wheel Mounting & Roller Configuration
Follow these exact physical checks before writing any software code. Mecanum wheels feature 45-degree rollers that push sideways when turned.

Look at the Robot from Above (Top View): The top rollers of all four wheels contacting the carpet MUST form an "X" shape across your robot chassis.
Top-Left Wheel: Rollers point diagonally toward the top-right.
Top-Right Wheel: Rollers point diagonally toward the top-left.
Bottom-Left Wheel: Rollers point diagonally toward the top-left.
Bottom-Right Wheel: Rollers point diagonally toward the top-right.
Step 2: Driver Station Hardware Configuration
Map the hardware ports on your REV Control Hub using the Driver Station Android app.
Connect the device that u are using to the wifi network with the SSID (Network Name): where the password is #Password1
Open the Driver Station App > Tap Settings (three dots) > Select Configure Robot.
Create a new configuration or edit your active configuration.
Select your Control Hub / Expansion Hub > Select Motors.
Enter the exact case-sensitive string names into the software configuration as listed below:
Physical Wheel Location
Config Motor Name
Control Hub Motor Port
Software Direction Setup
 
Front Left
frontLeft
Port 0
REVERSE
Back Left
backLeft
Port 1
REVERSE
Front Right
frontRight
Port 2
FORWARD
Back Right
backRight
Port 3
FORWARD

Step 3: Understanding Mecanum Math & Power Normalization
Mecanum movement calculates each motor's power by adding and subtracting three joystick inputs: Y (forward/backward), X (strafing left/right), and RX (turning yaw).
1. Kinematic Equations Breakdown
Front Left Power = Y + X + RX — Adding X pushes FL forward during right strafe.
Back Left Power = Y - X + RX — Subtracting X drives BL backward during right strafe.
Front Right Power = Y - X - RX — Subtracting X drives FR backward during right strafe.
Back Right Power = Y + X - RX — Adding X drives BR forward during right strafe.
2. Why Power Normalization is Critical
If a driver pushes full forward (Y = 1.0), full strafe (X = 1.0), and full turn (RX = 1.0), the raw front left calculation becomes 1.0 + 1.0 + 1.0 = 3.0. Since motor power caps at 1.0, unclipped values distort the drive angle. Normalization scales all 4 motor powers down proportionally so the maximum output magnitude is exactly 1.0.
Step 4: Robot-Centric TeleOp & Line-by-Line Code Explanation
Open http://192.168.43.1:8080 in your web browser, click OnBotJava, create a new file and name it a suitable name and paste the following complete code:
package org.firstinspires.ftc.teamcode;

import com.qualcomm.robotcore.eventloop.opmode.LinearOpMode;
import com.qualcomm.robotcore.eventloop.opmode.TeleOp;
import com.qualcomm.robotcore.hardware.DcMotor;
import com.qualcomm.robotcore.hardware.DcMotorSimple;

@TeleOp(name = "Mecanum Robot-Centric", group = "Linear OpMode")
public class MecanumTeleOpRobotCentric extends LinearOpMode {

    @Override
    public void runOpMode() throws InterruptedException {
        // Line A1: Fetch hardware motor objects from Driver Station configuration
        DcMotor frontLeftMotor = hardwareMap.get(DcMotor.class, "frontLeft");
        DcMotor backLeftMotor = hardwareMap.get(DcMotor.class, "backLeft");
        DcMotor frontRightMotor = hardwareMap.get(DcMotor.class, "frontRight");
        DcMotor backRightMotor = hardwareMap.get(DcMotor.class, "backRight");

        // Line A2: Reverse left side motors so positive power moves the robot forward
        frontLeftMotor.setDirection(DcMotorSimple.Direction.REVERSE);
        backLeftMotor.setDirection(DcMotorSimple.Direction.REVERSE);
        frontRightMotor.setDirection(DcMotorSimple.Direction.FORWARD);
        backRightMotor.setDirection(DcMotorSimple.Direction.FORWARD);

        // Line A3: Set active braking behavior when power drops to zero
        frontLeftMotor.setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE);
        backLeftMotor.setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE);
        frontRightMotor.setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE);
        backRightMotor.setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE);

        // Line A4: Send initialization status to Driver Station screen
        telemetry.addData("Status", "Initialized. Press Play.");
        telemetry.update();

        // Line A5: Pause execution until driver presses Play button on Driver Station
        waitForStart();

        // Line A6: Main driver control loop (executes repeatedly during TeleOp)
        while (opModeIsActive()) {
            // Line A7: Read joystick inputs and apply corrections
            double y = -gamepad1.left_stick_y; // Invert Y stick because pushing UP returns negative values
            double x = gamepad1.left_stick_x * 1.1; // Multiplier compensates for sideways roller scrub friction
            double rx = gamepad1.right_stick_x; // Yaw rotational turning control

            // Line A8: Compute raw power values for each wheel using Mecanum kinematics
            double frontLeftPower = y + x + rx;
            double backLeftPower = y - x + rx;
            double frontRightPower = y - x - rx;
            double backRightPower = y + x - rx;

            // Line A9: Find maximum absolute power value among all 4 wheels
            double maxPower = Math.max(Math.abs(frontLeftPower), Math.abs(backLeftPower));
            maxPower = Math.max(maxPower, Math.abs(frontRightPower));
            maxPower = Math.max(maxPower, Math.abs(backRightPower));

            // Line A10: Scale down powers proportionally if any single wheel exceeds 1.0
            if (maxPower > 1.0) {
                frontLeftPower /= maxPower;
                backLeftPower /= maxPower;
                frontRightPower /= maxPower;
                backRightPower /= maxPower;
            }

            // Line A11: Send physical motor power commands to hardware controllers
            frontLeftMotor.setPower(frontLeftPower);
            backLeftMotor.setPower(backLeftPower);
            frontRightMotor.setPower(frontRightPower);
            backRightMotor.setPower(backRightPower);

            // Line A12: Display live diagnostic data on Driver Station screen
            telemetry.addData("Inputs", "Y: %.2f | X: %.2f | RX: %.2f", y, x, rx);
            telemetry.addData("Powers", "FL: %.2f | BL: %.2f | FR: %.2f | BR: %.2f", 
                    frontLeftPower, backLeftPower, frontRightPower, backRightPower);
            telemetry.update();
        }
    }
}
Line-by-Line Breakdown of What Each Code Block Does:
Line Reference
Java Code Statement
Detailed Action & Purpose
 
@TeleOp(...)
OpMode Registration Annotation
Tells FTC SDK to list this class under TeleOp modes on the Driver Station drop-down menu.
Line A1
hardwareMap.get(DcMotor.class, "...")
Links the Java variable name to the physical motor port defined in your Driver Station configuration.
Line A2
setDirection(DcMotorSimple.Direction.REVERSE)
Reverses left side motors. Because left motors are mirrored physically, sending positive power without reversing would make the robot spin in place.
Line A3
setZeroPowerBehavior(ZeroPowerBehavior.BRAKE)
Forces motor coils to resist turning when power is zero, stopping coasting instantly when joysticks are released.
Line A5
waitForStart()
Blocks further execution while drivers press INIT. Code waits here until the driver presses the PLAY button.
Line A7
double y = -gamepad1.left_stick_y;
FTC Gamepads return -1.0 when pushed forward and +1.0 when pulled back. Reversing the sign ensures pushing forward drives the robot forward.
Line A7
double x = gamepad1.left_stick_x * 1.1;
Strafing encounters extra friction due to roller scrubbing. Scaling X by 1.1x balances lateral and forward speed response.
Line A8
frontLeftPower = y + x + rx;
Calculates Front Left motor power by summing axial (Y), lateral (X), and rotational (RX) control vectors.
Line A9-A10
if (maxPower > 1.0) { power /= maxPower; }
Divides all 4 wheel power outputs by the highest calculated magnitude if any motor exceeds 1.0, preserving steering ratios.
Line A11
frontLeftMotor.setPower(frontLeftPower);
Applies normalized float values (-1.0 to 1.0) to hardware motor speed controllers.

Step 5: Field-Centric TeleOp & IMU Integration 
Field-centric mode uses the REV Control Hub IMU gyro sensor so that pushing UP on the joystick always moves the robot forward regardless of the robot's orientation.
package org.firstinspires.ftc.teamcode;

import com.qualcomm.robotcore.eventloop.opmode.LinearOpMode;
import com.qualcomm.robotcore.eventloop.opmode.TeleOp;
import com.qualcomm.robotcore.hardware.DcMotor;
import com.qualcomm.robotcore.hardware.DcMotorSimple;
import com.qualcomm.hardware.rev.RevHubOrientationOnRobot;
import com.qualcomm.robotcore.hardware.IMU;
import org.firstinspires.ftc.robotcore.external.navigation.AngleUnit;

@TeleOp(name = "Mecanum Field-Centric", group = "Linear OpMode")
public class MecanumTeleOpFieldCentric extends LinearOpMode {

    @Override
    public void runOpMode() throws InterruptedException {
        DcMotor frontLeftMotor = hardwareMap.get(DcMotor.class, "frontLeft");
        DcMotor backLeftMotor = hardwareMap.get(DcMotor.class, "backLeft");
        DcMotor frontRightMotor = hardwareMap.get(DcMotor.class, "frontRight");
        DcMotor backRightMotor = hardwareMap.get(DcMotor.class, "backRight");

        frontLeftMotor.setDirection(DcMotorSimple.Direction.REVERSE);
        backLeftMotor.setDirection(DcMotorSimple.Direction.REVERSE);
        frontRightMotor.setDirection(DcMotorSimple.Direction.FORWARD);
        backRightMotor.setDirection(DcMotorSimple.Direction.FORWARD);

        // Line B1: Fetch internal REV Control Hub IMU hardware instance
        IMU imu = hardwareMap.get(IMU.class, "imu");
        
        // Line B2: Define physical mounting orientation of REV Hub on robot chassis
        IMU.Parameters parameters = new IMU.Parameters(new RevHubOrientationOnRobot(
                RevHubOrientationOnRobot.LogoFacingDirection.UP,
                RevHubOrientationOnRobot.UsbFacingDirection.FORWARD));
        
        // Line B3: Initialize IMU gyro sensors with specified orientation parameters
        imu.initialize(parameters);

        waitForStart();

        while (opModeIsActive()) {
            double y = -gamepad1.left_stick_y;
            double x = gamepad1.left_stick_x * 1.1;
            double rx = gamepad1.right_stick_x;

            // Line B4: Reset field zero reference heading when driver presses Options / Start button
            if (gamepad1.options || gamepad1.start) {
                imu.resetYaw();
            }

            // Line B5: Read current robot yaw orientation heading angle in radians
            double botHeading = imu.getRobotYawPitchRollAngles().getYaw(AngleUnit.RADIANS);

            // Line B6: Apply 2D inverse rotation transformation matrix to joystick vectors
            double rotX = x * Math.cos(-botHeading) - y * Math.sin(-botHeading);
            double rotY = x * Math.sin(-botHeading) + y * Math.cos(-botHeading);

            // Line B7: Calculate individual motor powers using field-rotated control vectors
            double frontLeftPower = rotY + rotX + rx;
            double backLeftPower = rotY - rotX + rx;
            double frontRightPower = rotY - rotX - rx;
            double backRightPower = rotY + rotX - rx;

            // Line B8: Normalize calculated motor powers
            double maxPower = Math.max(Math.abs(frontLeftPower), Math.abs(backLeftPower));
            maxPower = Math.max(maxPower, Math.abs(frontRightPower));
            maxPower = Math.max(maxPower, Math.abs(backRightPower));

            if (maxPower > 1.0) {
                frontLeftPower /= maxPower;
                backLeftPower /= maxPower;
                frontRightPower /= maxPower;
                backRightPower /= maxPower;
            }

            // Line B9: Apply power commands to physical motors
            frontLeftMotor.setPower(frontLeftPower);
            backLeftMotor.setPower(backLeftPower);
            frontRightMotor.setPower(frontRightPower);
            backRightMotor.setPower(backRightPower);
        }
    }
}
Line-by-Line Breakdown of Field-Centric IMU Mathematics:
Line Reference
Java Statement / Math Concept
Detailed Action & Purpose
 
Line B1-B3
RevHubOrientationOnRobot(...)
Configures pitch/roll/yaw axes relative to Control Hub placement (e.g., Logo pointing UP, USB ports facing FORWARD).
Line B4
imu.resetYaw()
Zeroes out current robot facing direction. Driver presses Options/Start button when facing forward to re-align field coordinate orientation.
Line B5
getYaw(AngleUnit.RADIANS)
Retrieves heading angle in radians required by standard Java trigonometric functions (Math.cos / Math.sin).
Line B6
rotX = x*cos(-θ) - y*sin(-θ)
rotY = x*sin(-θ) + y*cos(-θ)
2D Coordinate Rotation Matrix: Rotates stick vector (X,Y) backwards by negative heading angle (-), translating field reference frame into robot reference frame.

Step 7: Testing Sequence & Troubleshooting Guide
Bench Test (Robot on Blocks): Elevate the robot chassis so all 4 wheels spin freely off the ground. Initialize TeleOp and verify individual stick movements:
Left Stick UP → All 4 wheels spin forward.
Left Stick RIGHT → FL & BR spin forward; FR & BL spin backward.
Right Stick RIGHT → Left wheels spin forward; Right wheels spin backward.
On-Tile Testing & Diagnostics: Place the robot on the ground. If behavior is incorrect, follow the diagnostic troubleshooting steps below:
Observed Symptom
Underlying Cause
Step-by-Step Fix Action
 
Robot spins in place when attempting to strafe left/right
Wheels mounted backwards or motor reversed improperly.
Verify Step 1 top view forms an "X". Ensure frontLeft and backLeft are set to REVERSE.
Strafing direction is reversed (moves left when pushing stick right)
Inverted X vector sign.
In Line A7, change gamepad1.left_stick_x to -gamepad1.left_stick_x.
Robot drifts diagonally while trying to drive straight
Uneven center of mass or roller friction variance.
Rebalance battery/weight across chassis center or implement IMU heading PID stabilization.
Robot continues coasting after joystick release
Motors set to FLOAT mode.
Verify Line A3 sets setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE) on all 4 motors.


