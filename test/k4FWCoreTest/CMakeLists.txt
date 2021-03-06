################################################################################
# Package: k4FWCoreTest
################################################################################

find_package(EDM4HEP)




file(GLOB k4fwcoretest_plugin_sources src/components/*.cpp)
gaudi_add_module(k4FWCoreTestPlugins
                 SOURCES ${k4fwcoretest_plugin_sources}
                 LINK Gaudi::GaudiAlgLib Gaudi::GaudiKernel k4FWCore ROOT::Core ROOT::RIO ROOT::Tree EDM4HEP::edm4hepDict EDM4HEP::edm4hep)


include(CTest)

set(K4RUN ${PROJECT_SOURCE_DIR}/k4FWCore/scripts/k4run)

get_target_property(edm4hep_lib EDM4HEP::edm4hepDict LOCATION)
get_filename_component(edm4hep_loc ${edm4hep_lib} DIRECTORY)

get_target_property(root_lib ROOT::Core LOCATION)
get_filename_component(root_loc ${root_lib} DIRECTORY)

get_target_property(podio_lib podio::podio LOCATION)
get_filename_component(podio_loc ${podio_lib} DIRECTORY)

#--- The genConf directory has been renamed to genConfDir in Gaudi 35r1
#--- See https://gitlab.cern.ch/gaudi/Gaudi/-/merge_requests/1158
set(GAUDI_GENCONF_DIR "genConfDir")
if (${Gaudi_VERSION} VERSION_LESS 35.1)
  set(GAUDI_GENCONF_DIR "genConf")
endif()


function(set_test_env _testname)
  set_property(TEST ${_testname} APPEND PROPERTY ENVIRONMENT "ROOT_INCLUDE_PATH=${podio_loc}/../include/podio:${edm4hep_loc}/../include:$ENV{ROOT_INCLUDE_PATH}")
  set_property(TEST ${_testname} APPEND PROPERTY ENVIRONMENT "LD_LIBRARY_PATH=${CMAKE_BINARY_DIR}:${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}:${CMAKE_BINARY_DIR}/test/k4FWCoreTest:${root_loc}:${edm4hep_loc}:${podio_loc}:$ENV{LD_LIBRARY_PATH}")
  set_property(TEST ${_testname} APPEND PROPERTY ENVIRONMENT "PYTHONPATH=${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}/${GAUDI_GENCONF_DIR}:${CMAKE_BINARY_DIR}/test/k4FWCoreTest/${GAUDI_GENCONF_DIR}:$ENV{PYTHONPATH}")
endfunction()


add_test(NAME CreateExampleEventData
         WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
         COMMAND ${K4RUN} options/createExampleEventData.py)
set_test_env(CreateExampleEventData)

add_test(NAME TwoProducers
         WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
         COMMAND ${K4RUN} options/TwoProducers.py --filename output_k4fwcore_test_twoproducer2.root --magicNumberOffset.Producer2 12345 --Producer1.magicNumberOffset 54321 )
set_test_env(TwoProducers)

add_test(NAME ReadExampleEventData
         WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
         COMMAND ${K4RUN} options/readExampleEventData.py)
set_test_env(ReadExampleEventData)
set_property(TEST ReadExampleEventData APPEND PROPERTY DEPENDS CreateExampleEventData)

add_test(NAME ReadExampleDataFromNthEvent
         WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
         COMMAND ${K4RUN} options/readExampleDataFromNthEvent.py)
set_test_env(ReadExampleDataFromNthEvent)

add_test(NAME AlgorithmWithTFile
         WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
         COMMAND ${K4RUN} options/TestAlgorithmWithTFile.py)
set_test_env(AlgorithmWithTFile)

add_test(NAME AlgorithmWithTFileCheckFrameworkOutput
         WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
         COMMAND  python scripts/check_TestAlgorithmWithTFile_framework_nonempty.py)
set_property(TEST AlgorithmWithTFileCheckFrameworkOutput APPEND PROPERTY DEPENDS AlgorithmWithTFile)
set_test_env(AlgorithmWithTFileCheckFrameworkOutput)

add_test(NAME AlgorithmWithTFileCheckMyTFileOutput
         WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
         COMMAND  python scripts/check_TestAlgorithmWithTFile_myTFile_nonempty.py)
set_property(TEST AlgorithmWithTFileCheckMyTFileOutput APPEND PROPERTY DEPENDS AlgorithmWithTFile)
set_test_env(AlgorithmWithTFileCheckMyTFileOutput)

add_test(NAME CheckCommandLineArguments
         WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
         COMMAND ${K4RUN} options/createHelloWorld.py --HelloWorldAlg1.PerEventPrintMessage TwasBrilligAndTheSlithyToves
)
set_tests_properties(CheckCommandLineArguments PROPERTIES
  PASS_REGULAR_EXPRESSION "TwasBrilligAndTheSlithyToves"
)
set_test_env(CheckCommandLineArguments)

add_test(NAME checkKeepDropSwitch
         WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
         COMMAND python scripts/check_KeepDropSwitch.py)
set_test_env(checkKeepDropSwitch)
set_property(TEST checkKeepDropSwitch APPEND PROPERTY DEPENDS ReadExampleEventData)

add_test(NAME TestDataHandleUniquePtr
         WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
         COMMAND ${K4RUN} options/TestDataHandleUniquePtr.py)
set_test_env(TestDataHandleUniquePtr)


