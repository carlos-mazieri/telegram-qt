
set(telegram_qt_SOURCES
    TelegramNamespace.cpp
    CAppInformation.cpp
    CTelegramCore.cpp
    CTelegramDispatcher.cpp
    CTelegramConnection.cpp
    CTelegramStream.cpp
    CTcpTransport.cpp
    CRawStream.cpp
    Utils.cpp
    TLValues.cpp
)

set(telegram_qt_META_HEADERS
    TelegramNamespace.hpp
    CTelegramCore.hpp
    CTelegramDispatcher.hpp
    CTelegramConnection.hpp
    CTelegramTransport.hpp
    CTcpTransport.hpp
    TLValues.hpp
)

set(telegram_qt_HEADERS
    telegramqt_export.h
    TelegramNamespace.hpp
    CAppInformation.hpp
    CTelegramCore.hpp
    CTelegramDispatcher.hpp
    CTelegramConnection.hpp
    CTelegramStream.hpp
    CTelegramTransport.hpp
    CTcpTransport.hpp
    CRawStream.hpp
    Utils.hpp
    TLValues.hpp
    TLTypes.hpp
    crypto-rsa.hpp
    crypto-aes.hpp
)

set(telegram_qt_public_HEADERS
    telegramqt_export.h
    CAppInformation.hpp
    TelegramNamespace.hpp
    CTelegramCore.hpp
)

include_directories(
    ${CMAKE_SOURCE_DIR}
    ${CMAKE_BINARY_DIR}
    ${OPENSSL_INCLUDE_DIR}
    ${ZLIB_INCLUDE_DIR}
)

if (DEVELOPER_BUILD)
    list(APPEND telegram_qt_SOURCES TLTypesDebug.cpp)
    list(APPEND telegram_qt_HEADERS TLTypesDebug.hpp)
    add_definitions(-DDEVELOPER_BUILD)
endif()

add_definitions(-DTELEGRAMQT_LIBRARY)

if (USE_QT4)
    QT4_WRAP_CPP(telegram_qt_MOC_SOURECES ${telegram_qt_META_HEADERS})
else()
    QT5_WRAP_CPP(telegram_qt_MOC_SOURECES ${telegram_qt_META_HEADERS})
endif()

if (STATIC_BUILD)
    add_library(telegram-qt${QT_VERSION_MAJOR} STATIC ${telegram_qt_SOURCES} ${telegram_qt_MOC_SOURECES})
else()
    add_library(telegram-qt${QT_VERSION_MAJOR} SHARED ${telegram_qt_SOURCES} ${telegram_qt_MOC_SOURECES})
endif()

target_include_directories(telegram-qt${QT_VERSION_MAJOR} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

if(USE_QT4)
    target_link_libraries(telegram-qt${QT_VERSION_MAJOR}
        ${QT_QTCORE_LIBRARY}
        ${QT_QTNETWORK_LIBRARY}
    )
    target_include_directories(telegram-qt${QT_VERSION_MAJOR} PRIVATE
        ${QT_INCLUDES}
    )
else()
    target_link_libraries(telegram-qt${QT_VERSION_MAJOR}
        Qt5::Core
        Qt5::Network
    )
    target_include_directories(telegram-qt${QT_VERSION_MAJOR} PRIVATE
        ${QT5_INCLUDES}
    )
endif()

target_link_libraries(telegram-qt${QT_VERSION_MAJOR}
    ${OPENSSL_LIBRARIES}
    ${ZLIB_LIBRARIES}
)

set(TELEGRAM_QT_INCLUDE_DIR ${CMAKE_INSTALL_INCLUDEDIR}/telegram-qt${QT_VERSION_MAJOR})
set(TELEGRAM_QT_LIB_DIR ${CMAKE_INSTALL_LIBDIR})

# Set the correct version number
set_target_properties(
    telegram-qt${QT_VERSION_MAJOR} PROPERTIES
    SOVERSION ${TELEGRAM_QT_ABI}
    VERSION ${TELEGRAM_QT_VERSION}
)

install(
    TARGETS telegram-qt${QT_VERSION_MAJOR}
    LIBRARY DESTINATION ${TELEGRAM_QT_LIB_DIR}
    ARCHIVE DESTINATION ${TELEGRAM_QT_LIB_DIR}
    COMPONENT Library
)

install(
    DIRECTORY TelegramQt
    DESTINATION ${TELEGRAM_QT_INCLUDE_DIR}
    COMPONENT Devel
)

install(
    FILES ${telegram_qt_public_HEADERS}
    DESTINATION ${TELEGRAM_QT_INCLUDE_DIR}/TelegramQt
    COMPONENT Devel
)

# CMake find package stuff
include(CMakePackageConfigHelpers)

configure_file(TelegramQtConfig.cmake.in "${CMAKE_CURRENT_BINARY_DIR}/TelegramQt${QT_VERSION_MAJOR}Config.cmake" @ONLY)
write_basic_package_version_file("${CMAKE_CURRENT_BINARY_DIR}/TelegramQt${QT_VERSION_MAJOR}ConfigVersion.cmake"
                                    VERSION ${TELEGRAM_QT_VERSION}
                                    COMPATIBILITY ExactVersion)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/TelegramQt${QT_VERSION_MAJOR}ConfigVersion.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/TelegramQt${QT_VERSION_MAJOR}Config.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/TelegramQt${QT_VERSION_MAJOR}
    COMPONENT Devel
)
