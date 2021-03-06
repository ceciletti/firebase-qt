# firebase-qt
Qt Wrapper for Firebase C++ API

Supports:
 * Messaging for Push Notifications
 * Phone Authentication

## Example of usage

Google Firebase SDK use futures that runs on different threads, so in order to avoid a crash due updating a Qt GUI from the wrong thread always use Qt:: QueuedConnection.

Initialize:

    auto firebase = new FirebaseQtApp(this);
    auto messaging = new FirebaseQtMessaging(firebase);
    connect(messaging, &FirebaseQtMessaging::tokenReceived, this, &Central::firebaseUpdateMessagingToken, Qt::QueuedConnection);
    connect(messaging, &FirebaseQtMessaging::messageReceived, this, &Central::firebaseOnMessage, Qt::QueuedConnection);

    m_firebaseAuth = new FirebaseQtAuth(firebase);
    connect(m_firebaseAuth, &FirebaseQtAuth::signInToken, this, &Central::firebaseUserToken, Qt::QueuedConnection);
    connect(m_firebaseAuth, &FirebaseQtAuth::signInError, this, &Central::firebaseUserTokenError, Qt::QueuedConnection);

    m_firebaseAuthPhone = new FirebaseQtAuthPhone(m_firebaseAuth);
    connect(m_firebaseAuthPhone, &FirebaseQtAuthPhone::codeSent, this, &Central::firebaseAuthCodeSent, Qt::QueuedConnection);
    connect(m_firebaseAuthPhone, &FirebaseQtAuthPhone::verificationFailed, this, &Central::firebaseAuthFailed, Qt::QueuedConnection);
    connect(m_firebaseAuthPhone, &FirebaseQtAuthPhone::verificationCompleted, this, &Central::firebaseAuthCompleted, Qt::QueuedConnection);
    
    firebase->initialize();
    
    
Ask for a phone number authentication:

    m_firebaseAuthPhone->verifyPhoneNumber("+55999999999");

## Compiling

This is the most chalenging part, especially on iOS, because qmake doesn't support CocoaPods, and I'm not sure how mature Qt CMake integration is for iOS, so you have to download the C++ SDK **and** the Firebase iOS framework, here is how my qmake looks like (I'm pretty sure I can omit some links in iOS but didn't have the time to check):

        QT += quick svg
    CONFIG += c++11
    TARGET = 'my-app'

    android: {
        QT += androidextras
    }
    android|ios {
        CONFIG += qtquickcompiler
    }

    # The following define makes your compiler emit warnings if you use
    # any Qt feature that has been marked deprecated (the exact warnings
    # depend on your compiler). Refer to the documentation for the
    # deprecated API to know how to port your code away from it.
    DEFINES += QT_DEPRECATED_WARNINGS

    # You can also make your code fail to compile if it uses deprecated APIs.
    # In order to do so, uncomment the following line.
    # You can also select to disable deprecated APIs only up to a certain version of Qt.
    #DEFINES += QT_DISABLE_DEPRECATED_BEFORE=0x060000    # disables all the APIs deprecated before Qt 6.0.0

    SOURCES += \
        main.cpp

    android|ios {
        SOURCES += \
            firebaseqtapp.cpp \
            firebaseqtauth.cpp \
            firebaseqtauthcredential.cpp \
            firebaseqtmessaging.cpp \
            firebaseqtauthphone.cpp \
            firebaseqtabstractmodule.cpp
    }

    RESOURCES += qml.qrc

    # Additional import path used to resolve QML modules in Qt Creator's code model
    QML_IMPORT_PATH =

    # Additional import path used to resolve QML modules just for Qt Quick Designer
    QML_DESIGNER_IMPORT_PATH =

    # Default rules for deployment.
    qnx: target.path = /tmp/$${TARGET}/bin
    else: unix:!android: target.path = /opt/$${TARGET}/bin
    !isEmpty(target.path): INSTALLS += target

    android|ios {
        HEADERS += \
            firebaseqtabstractmodule.h \
            firebaseqtapp.h \
            firebaseqtapp_p.h \
            firebaseqtauth.h \
            firebaseqtauth_p.h \
            firebaseqtauthcredential.h \
            firebaseqtauthcredential_p.h \
            firebaseqtmessaging.h \
            firebaseqtauthphone.h
    }

    DISTFILES += \
        android/AndroidManifest.xml \
        android/gradle/wrapper/gradle-wrapper.jar \
        android/gradlew \
        android/res/values/libs.xml \
        android/build.gradle \
        android/gradle/wrapper/gradle-wrapper.properties \
        android/gradlew.bat \
        android/google-services.json

    CLIENT_DIR = my-app

    GOOGLE_FIREBASE_SDK = $$PWD/../firebase_cpp_sdk
    GOOGLE_IOS_FIREBASE_SDK = $$PWD/../Firebase

    INCLUDEPATH += $${GOOGLE_FIREBASE_SDK}/include
    DEPENDPATH += $${GOOGLE_FIREBASE_SDK}/include

    contains(ANDROID_TARGET_ARCH,armeabi-v7a) {
        ANDROID_PACKAGE_SOURCE_DIR = \
            $$PWD/android

        ANDROID_EXTRA_LIBS = \
        $$PWD/../android_openssl/arm/libcrypto_1_1.so \
        $$PWD/../android_openssl/arm/libssl_1_1.so

        LIBS += -L$${GOOGLE_FIREBASE_SDK}/libs/android/armeabi-v7a/c++/ -lfirebase_app -lfirebase_messaging -lfirebase_auth

        PRE_TARGETDEPS += $${GOOGLE_FIREBASE_SDK}/libs/android/armeabi-v7a/c++/libfirebase_app.a
        PRE_TARGETDEPS += $${GOOGLE_FIREBASE_SDK}/libs/android/armeabi-v7a/c++/libfirebase_messaging.a
        PRE_TARGETDEPS += $${GOOGLE_FIREBASE_SDK}/libs/android/armeabi-v7a/c++/libfirebase_auth.a
    }

    contains(ANDROID_TARGET_ARCH,arm64-v8a) {
        ANDROID_PACKAGE_SOURCE_DIR = \
            $$PWD/android

        ANDROID_EXTRA_LIBS = \
        $$PWD/../android_openssl/arm64/libssl_1_1.so \
        $$PWD/../android_openssl/arm64/libcrypto_1_1.so

        LIBS += -L$${GOOGLE_FIREBASE_SDK}/libs/android/arm64-v8a/c++/ -lfirebase_app -lfirebase_messaging -lfirebase_auth

        PRE_TARGETDEPS += $${GOOGLE_FIREBASE_SDK}/libs/android/arm64-v8a/c++/libfirebase_app.a
        PRE_TARGETDEPS += $${GOOGLE_FIREBASE_SDK}/libs/android/arm64-v8a/c++/libfirebase_messaging.a
        PRE_TARGETDEPS += $${GOOGLE_FIREBASE_SDK}/libs/android/arm64-v8a/c++/libfirebase_auth.a
    }

    ios: {
        GOOGLE_FIREBASE_SDK_LIBS_PATH = $${GOOGLE_FIREBASE_SDK}/libs/ios/universal/
        GOOGLE_FIREBASE_SDK_FW_PATH = $${GOOGLE_FIREBASE_SDK}/frameworks/ios/universal/
        PRODUCT_NAME = 'my-app'
        QMAKE_APPLE_TARGETED_DEVICE_FAMILY = 1

        QMAKE_ASSET_CATALOGS = $$CLIENT_DIR/ios/Images.xcassets
        QMAKE_ASSET_CATALOGS_APP_ICON = "AppIcon"

        MY_ENTITLEMENTS.name = CODE_SIGN_ENTITLEMENTS
        MY_ENTITLEMENTS.value = $$PWD/$$CLIENT_DIR/ios/my-app.entitlements
        QMAKE_MAC_XCODE_SETTINGS += MY_ENTITLEMENTS

        app_launch_images.files = icons/logo.jpeg $$CLIENT_DIR/ios/splash/Launch.xib $$files($$CLIENT_DIR/ios/splash/LaunchImage*.png)
        QMAKE_BUNDLE_DATA += app_launch_images

        QMAKE_INFO_PLIST = $$CLIENT_DIR/ios/Info.plist

        DISTFILES += \
            $$CLIENT_DIR/ios/Info.plist \
            $$CLIENT_DIR/ios/GoogleService-Info.plist \

        # You must deploy your Google Play config file
        deployment.files = $$CLIENT_DIR/ios/GoogleService-Info.plist
        deployment.path =
        QMAKE_BUNDLE_DATA += deployment

        QMAKE_LFLAGS += -ObjC

        LIBS += \
            -ObjC \
            -lsqlite3 \
            -lz \
            -L$${GOOGLE_FIREBASE_SDK_LIBS_PATH} \
            -framework MediaPlayer \
            -framework CoreMotion \
            -framework CoreTelephony \
            -framework MessageUI \
            -framework GLKit \
            -framework AddressBook \
            -framework CoreFoundation \
            -framework Foundation \
            -framework Security \
            -framework UIKit \
            -framework SystemConfiguration \
            -framework GSS

        LIBS +=  \
                -framework StoreKit \
                -F$${GOOGLE_IOS_FIREBASE_SDK}/Analytics \
                -framework FirebaseAnalytics \
                -framework FirebaseCore \
                -framework FirebaseCoreDiagnostics \
                -framework FirebaseInstanceID \
                -framework GoogleDataTransport \
                -framework GoogleDataTransportCCTSupport \
                -framework GoogleAppMeasurement \
                -framework GoogleUtilities \
                -framework nanopb

        LIBS +=  \
        -F$${GOOGLE_IOS_FIREBASE_SDK}/Messaging \
        -framework FirebaseMessaging \
        -framework Protobuf \
        -framework UserNotifications

        LIBS +=  \
        -F$${GOOGLE_FIREBASE_SDK_FW_PATH} \
        -framework firebase_messaging \
        \

        LIBS +=  \
        -F$${GOOGLE_IOS_FIREBASE_SDK}/Auth \
        -framework FirebaseAuth \
        -framework GTMSessionFetcher \
        -framework SafariServices

        LIBS +=  \
        -F$${GOOGLE_FIREBASE_SDK_FW_PATH} \
        -framework firebase_auth \
        -framework firebase \
        \

    }
