Tutorial Flutter LocalNotification dengan Firebase Cloud Messaging (FCM)
Packages:
https://pub.dev/packages/alarm
https://pub.dev/packages/flutter_local_notifications
Create flutter projects

mkdir tutorial_notifications
cd tutorial_notifications
flutter create .
2. Atur android/app/build.gradle menjadi:
compileSdkVersion 33
minSdkVersion 26
targetSdkVersion 33
3. Tambahkan package ini:
flutter pub add flutter_local_notifications firebase_core firebase_auth firebase_messaging
4. Tambahkan file app_icon.png ke lokasi ini:
android/app/src/main/res/drawable/app_icon.png
Silahkan gunakan gambarmu sendiri.
Atau download salah satu icon disini:
https://www.iconarchive.com/tag/owl
Bisa juga dengan menjalankan perintah CURL ini utk icon sementara:
curl https://i.ibb.co/vPQfPDJ/app-icon.png -o android/app/src/main/res/drawable/app_icon.png
5. Next, ubah AndroidManifest.xml, di lokasi ini:
android/app/src/main/AndroidManifest.xml
Tambahkan kode ini sebelum tag <application> </application>
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<uses-permission android:name="android.permission.VIBRATE" />
<uses-permission android:name="android.permission.USE_FULL_SCREEN_INTENT" />
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
6. Masih di AndroidManifest.xml, tambahkan kode ini diantara <application> dan </application>
<service
    android:name="dev.fluttercommunity.plus.androidalarmmanager.AlarmService"
    android:permission="android.permission.BIND_JOB_SERVICE"
    android:exported="false"/>
<receiver
    android:name="dev.fluttercommunity.plus.androidalarmmanager.AlarmBroadcastReceiver"
    android:exported="false"/>
<receiver
    android:name="dev.fluttercommunity.plus.androidalarmmanager.RebootBroadcastReceiver"
    android:enabled="false"
    android:exported="false">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
7. Setup Firebase sampai Terhubung:
Ikuti panduan disini:
https://medium.com/@denyocr.world/step-by-step-cara-setup-flutter-firebase-713f9187262b
Atau jika kamu sudah paham,
Langsung saja eksekusi:
flutterfire configure
Lanjutkan sampai project Flutter terhubung dengan Firebase.
Ditandai dengan kamu sudah mengatur file main.dart
import 'firebase_options.dart';
//...

void main() async {
  //----------------
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  //----------------
  runApp(const MyApp());
}
8. Buat file ini:
lib/service/notification_service/notification_service.dart
Isi dengan kode ini:
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter/foundation.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import '../../firebase_options.dart';

@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  print('Handling a background message ${message.messageId}');
}

class NotificationService {
  static late FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin;
  static late AndroidNotificationChannel channel;
  static bool isFlutterLocalNotificationsInitialized = false;

  initNotifications() async {
    FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);
    FirebaseMessaging.onMessage.listen(showFlutterNotification);
    if (!kIsWeb) {
      await setupFlutterNotifications();
    }
  }

  Future<String?> getToken() async {
    String? token = await FirebaseMessaging.instance.getToken(
        vapidKey:
            'BNKkaUWxyP_yC_lki1kYazgca0TNhuzt2drsOrL6WrgGbqnMnr8ZMLzg_rSPDm6HKphABS0KzjPfSqCXHXEd06Y');

    print("FCM Token: $token");
    return token;
  }

  Future<void> setupFlutterNotifications() async {
    if (isFlutterLocalNotificationsInitialized) {
      return;
    }
    channel = const AndroidNotificationChannel(
      'high_importance_channel',
      'High Importance Notifications',
      description: 'This channel is used for important notifications.',
      importance: Importance.high,
    );
    flutterLocalNotificationsPlugin = FlutterLocalNotificationsPlugin();
    await flutterLocalNotificationsPlugin
        .resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>()
        ?.createNotificationChannel(channel);
    await FirebaseMessaging.instance
        .setForegroundNotificationPresentationOptions(
      alert: true,
      badge: true,
      sound: true,
    );
    isFlutterLocalNotificationsInitialized = true;
  }

  void showFlutterNotification(RemoteMessage message) {
    RemoteNotification? notification = message.notification;
    AndroidNotification? android = message.notification?.android;
    if (notification != null && android != null && !kIsWeb) {
      flutterLocalNotificationsPlugin.show(
        notification.hashCode,
        notification.title,
        notification.body,
        NotificationDetails(
          android: AndroidNotificationDetails(
            channel.id,
            channel.name,
            channelDescription: channel.description,
            icon: 'launch_background',
          ),
        ),
      );
    }
  }
}
9. Tambahkan kode ini di main.dart
if (await Permission.notification.request().isGranted) {
  await NotificationService().initNotifications();
  await NotificationService().getToken();
}
Seharusnya akan terlihat seperti ini:
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  //---
  await NotificationService().initNotifications();
  await NotificationService().getToken();
  //---
  runApp(const MyApp());
}
10. Jalankan Aplikasi, maka kamu dapat melihat fcm token yang muncul di console. Copy FCM token itu untuk mencoba notification-nya nanti.
Test Notification melalui menu Messaging pada Firebase Console
Masuk ke Firebase Console
Klik Messaging, dan klik Create your first campaign

Pilih Firebase Notifications Messages

3. Pilih Notifications
4. Isi Notifications title dan Notifications text, dan klik Send test message
5. Tambahkan FCM Token yang kamu dapatkan di Flutter dan klik icon plus. Lalu klik tombol Test
6. Sampai disini, seharusnya notifikasinya akan muncul di devicemu.
Contoh:
Jika tidak muncul, berarti ada tahapan yang kamu lewatkan,
Pastikan semua tahapannya sudah benar!
Masalah Umum Notifications
Notifications muncul double
Dugaan saat ini, ini muncul jika kamu menggunakan Emulator.
Tidak perlu ada perbaikan khusus, cukup menjalankan project Flutter-nya di Real Device.
Menggunakan Real Device adalah best practices jika kamu masih belajar menggunakan Flutter.
Sepengalaman saya, banyak masalah yang mungkin kamu alami selama menggunakan emulator. Daripada kamu mengatasi masalah itu dan menghabiskan waktu karenanya, mari kita gunakan real device saja.
Dan jika ingin di mirror, bisa menggunakan Vysor atau scrcpy