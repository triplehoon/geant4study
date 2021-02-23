# ExampleB4d 정리

## 헤더 선언

```CPP
#include "B4dDetectorConstruction.hh"
#include "B4dActionInitialization.hh"
#include "B4Analysis.hh"

#include "G4RunManagerFactory.hh"

#include "G4UImanager.hh"
#include "FTFP_BERT.hh"
#include "G4VisExecutive.hh"
#include "G4UIExecutive.hh"
#include "Randomize.hh"
#include "G4TScoreNtupleWriter.hh"

```

## 프로그램 사용법 설명 함수 ```PrintUsage()```
```CPP
//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo......

namespace {
  void PrintUsage() {
    G4cerr << " Usage: " << G4endl;
    G4cerr << " exampleB4d [-m macro ] [-u UIsession] [-t nThreads]" << G4endl;
    G4cerr << "   note: -t option is available only for multi-threaded mode."
           << G4endl;
  }
}
```

## Argument 관련 정리
```CPP
int main(int argc,char** argv)
{  
  // Evaluate arguments
  //
  if ( argc > 7 ) {
    PrintUsage();
    return 1;
  }
  
  G4String macro;
  G4String session;
#ifdef G4MULTITHREADED //멀티쓰레딩 사용시 nThread 변수 선언
  G4int nThreads = 0;
#endif
  for ( G4int i=1; i<argc; i=i+2 ) {
    if      ( G4String(argv[i]) == "-m" ) macro = argv[i+1];
    else if ( G4String(argv[i]) == "-u" ) session = argv[i+1];
#ifdef G4MULTITHREADED
    else if ( G4String(argv[i]) == "-t" ) {
      nThreads = G4UIcommand::ConvertToInt(argv[i+1]);
    }
#endif
    else {
      PrintUsage();
      return 1;
    }
  }  

  ```

  ## runManager 선언 및 멀티쓰레딩 설정
  ```CPP
  // Detect interactive mode (if no macro provided) and define UI session
  //
  G4UIExecutive* ui = nullptr;
  if ( ! macro.size() ) {
    ui = new G4UIExecutive(argc, argv, session);
  }
  // Optionally: choose a different Random engine...
  //
  // G4Random::setTheEngine(new CLHEP::MTwistEngine);
  
  // Construct the MT run manager
  //

  auto* runManager =
    G4RunManagerFactory::CreateRunManager(G4RunManagerType::Default);
#ifdef G4MULTITHREADED
  if ( nThreads > 0 ) {
    runManager->SetNumberOfThreads(nThreads);
  }
#endif
```

##  필수 클래스 초기화
### Detectorconsruction 선언
```cpp
  // Set mandatory initialization classes
  //
  auto detConstruction = new B4dDetectorConstruction();
  runManager->SetUserInitialization(detConstruction);
```
### Pysicslist 선언 (FTFP_BERT)
```CPP
  auto physicsList = new FTFP_BERT;
  runManager->SetUserInitialization(physicsList);
```
### Action 선언
```CPP
  auto actionInitialization = new B4dActionInitialization();
  runManager->SetUserInitialization(actionInitialization);
```
---
## Visualization 선언
```CPP
  // Initialize visualization
  auto visManager = new G4VisExecutive;
  // G4VisExecutive can take a verbosity argument - see /vis/verbose guidance.
  // G4VisManager* visManager = new G4VisExecutive("Quiet");
  visManager->Initialize();

  // Get the pointer to the User Interface manager
  auto UImanager = G4UImanager::GetUIpointer();
```
## ntuplewriter 활성화
>An ntuple with three columns is created for each primitive scorer:
int column - eventNumber 
int column - copyNumber
double column - scored value

클래스 타고 들어가니깐, 라는데 정확히는 모르겠습니다. 아마 각 스코어링 마다 정보를 표시해 주는 기능을 활성화 시키는 듯합니다.
```CPP
  // Activate score ntuple writer
  // The Root output type (Root) is selected in B3Analysis.hh.
  G4TScoreNtupleWriter<G4AnalysisManager> scoreNtupleWriter;
  // The verbose level can be set via UI commands
  // /score/ntuple/writerVerbose level
  // or via the score ntuple writer function:
  // scoreNtupleWriter.SetVerboseLevel(1);
```

## UI session 및 매크로 설정
```CPP
  // Process macro or start UI session
  //
  if ( macro.size() ) {
    // batch mode
    G4String command = "/control/execute ";
    UImanager->ApplyCommand(command+macro);
  }
  else  {  
    // interactive mode : define UI session
    UImanager->ApplyCommand("/control/execute init_vis.mac");
    if (ui->IsGUI()) {
      UImanager->ApplyCommand("/control/execute gui.mac");
    }
    ui->SessionStart();
    delete ui;
  }
```

## 동적할당 해제
```CPP
  // Job termination
  // Free the store: user actions, physics_list and detector_description are
  // owned and deleted by the run manager, so they should not be deleted 
  // in the main() program !

  delete visManager;
  delete runManager;
}

//....oooOO0OOooo........oooOO0OOooo........oooOO0OOooo........oooOO0OOooo.....
```