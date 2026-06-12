# SonicN — Symbian HLE worklist (imports)

Every symbol `sonicn.app` imports, grouped by source DLL. This is the **exact** set of
Symbian functions the native runtime must provide (implement or stub). Dumped by IDA
Professional 9.1 (EPOC loader) — ordinals already demangled to SDK signatures.

**Total: 233 imports across 11 DLLs.**

Status legend: ⬜ not started · 🟨 stubbed · 🟩 implemented


## EUSER (78)
_Kernel/user lib: heap, descriptors, cleanup stack, active scheduler, math, timers, compiler runtime._

- ⬜ `Add__16CActiveSchedulerP7CActive`
- ⬜ `After__4UserG27TTimeIntervalMicroSeconds32`
- ⬜ `After__6RTimerR14TRequestStatusG27TTimeIntervalMicroSeconds32`
- ⬜ `AllocL__4Useri`
- ⬜ `Cancel__6RTimer`
- ⬜ `Cancel__7CActive`
- ⬜ `Close__11RHandleBase`
- ⬜ `CreateLocal__6RTimer`
- ⬜ `Exit__4Useri`
- ⬜ `FillZ__3MemPvi`
- ⬜ `Format__6TDes16Gt11TRefByValue1ZC7TDesC16e`
- ⬜ `GetTInt__C6TInt64`
- ⬜ `HomeTime__5TTime`
- ⬜ `Language__4User`
- ⬜ `LeaveIfError__4Useri`
- ⬜ `Move__5TRectii`
- ⬜ `NewL__9CPeriodici`
- ⬜ `Panic__4UserRC7TDesC16i`
- ⬜ `PopAndDestroy__12CleanupStack`
- ⬜ `PopAndDestroy__12CleanupStacki`
- ⬜ `Pop__12CleanupStack`
- ⬜ `PtrZ__5TDes8`
- ⬜ `PushL__12CleanupStackP5CBase`
- ⬜ `Rand__4MathR6TInt64`
- ⬜ `RunError__7CActivei`
- ⬜ `SetActive__7CActive`
- ⬜ `SetLength__5TDes8i`
- ⬜ `Sin__4MathRdRCd`
- ⬜ `Start__9CPeriodicG27TTimeIntervalMicroSeconds32T1G9TCallBack`
- ⬜ `TickCount__4User`
- ⬜ `Trap__5TTrapRi`
- ⬜ `UnTrap__5TTrap`
- ⬜ `_._5CBase`
- ⬜ `_._7CActive`
- ⬜ `__10TBufBase16i`
- ⬜ `__5CBase`
- ⬜ `__5TPtr8PUci`
- ⬜ `__5TPtr8PUcii`
- ⬜ `__5TRectRC5TSize`
- ⬜ `__5TRectiiii`
- ⬜ `__6TInt64i`
- ⬜ `__6TPtr16PUsi`
- ⬜ `__6TPtr16PUsii`
- ⬜ `__7CActivei`
- ⬜ `__7TPtrC16PCUs`
- ⬜ `__9TBufBase8i`
- ⬜ `__adddf3`
- ⬜ `__addsf3`
- ⬜ `__builtin_delete`
- ⬜ `__builtin_vec_delete`
- ⬜ `__builtin_vec_new`
- ⬜ `__divdf3`
- ⬜ `__divsi3`
- ⬜ `__dv__C6TInt64RC6TInt64`
- ⬜ `__fixdfsi`
- ⬜ `__fixsfsi`
- ⬜ `__floatsidf`
- ⬜ `__floatsisf`
- ⬜ `__gedf2`
- ⬜ `__gtdf2`
- ⬜ `__gtsf2`
- ⬜ `__ledf2`
- ⬜ `__lesf2`
- ⬜ `__ltdf2`
- ⬜ `__modsi3`
- ⬜ `__muldf3`
- ⬜ `__mulsf3`
- ⬜ `__ne__C6TInt64RC6TInt64`
- ⬜ `__nedf2`
- ⬜ `__negdf2`
- ⬜ `__nw__5CBaseUi`
- ⬜ `__pure_virtual`
- ⬜ `__subdf3`
- ⬜ `__subsf3`
- ⬜ `__udivsi3`
- ⬜ `memcpy`
- ⬜ `memset`
- ⬜ `newL__5CBaseUi`

## EIKCORE (55)
_S60 application framework (Eikon core)._

- ⬜ `ApplicationRect__C9CEikAppUi`
- ⬜ `BitmapStoreName__C15CEikApplication`
- ⬜ `CancelTrigger__13EikBubbleHelp`
- ⬜ `Capability__C15CEikApplicationR5TDes8`
- ⬜ `CheckHotKeyNotDimmedL__16MEikMenuObserveri`
- ⬜ `ConstructL__9CEikAppUi`
- ⬜ `CreateCustomCommandControlL__19MEikCommandObserveri`
- ⬜ `CreateDocumentL__15CEikApplicationP11CApaProcess`
- ⬜ `CreateFileL__9CEikAppUiRC7TDesC16`
- ⬜ `CreateFileStoreLC__12CEikDocumentR3RFsRC7TDesC16`
- ⬜ `DynInitMenuBarL__16MEikMenuObserveriP11CEikMenuBar`
- ⬜ `DynInitMenuPaneL__16MEikMenuObserveriP12CEikMenuPane`
- ⬜ `EIKCORE_290`
- ⬜ `EditL__12CEikDocumentP23MApaEmbeddedDocObserveri`
- ⬜ `Exit__9CEikAppUi`
- ⬜ `ExternalizeL__C12CEikDocumentR12RWriteStream`
- ⬜ `GetDefaultDocumentFileName__C15CEikApplicationRt4TBuf1i256`
- ⬜ `HandleApplicationSpecificEventL__9CEikAppUiiRC8TWsEvent`
- ⬜ `HandleAttemptDimmedSelectionL__16MEikMenuObserveri`
- ⬜ `HandleCommandL__9CEikAppUii`
- ⬜ `HandleMessageL__9CEikAppUiUlG4TUidRC6TDesC8`
- ⬜ `HandleModelChangeL__9CEikAppUi`
- ⬜ `HandleResourceChangeL__9CEikAppUii`
- ⬜ `HandleSideBarMenuL__9CEikAppUiiRC6TPointiPC15CEikHotKeyTable`
- ⬜ `HasChanged__C12CEikDocument`
- ⬜ `IsEmpty__C12CEikDocument`
- ⬜ `NewDocumentL__12CEikDocument`
- ⬜ `OfferKeyToAppL__16MEikMenuObserverRC9TKeyEvent10TEventCode`
- ⬜ `OpenAppInfoFileLC__C15CEikApplication`
- ⬜ `OpenFileL__9CEikAppUiRC7TDesC16`
- ⬜ `PrintL__12CEikDocumentRC12CStreamStore`
- ⬜ `ProcessCommandParametersL__9CEikAppUi11TApaCommandRt4TBuf1i256RC6TDesC8`
- ⬜ `ProcessMessageL__9CEikAppUiG4TUidRC6TDesC8`
- ⬜ `Reserved_1_MenuObserver__16MEikMenuObserver`
- ⬜ `Reserved_1__12CEikDocument`
- ⬜ `Reserved_1__15CEikApplication`
- ⬜ `Reserved_1__9CEikAppUi`
- ⬜ `Reserved_2__12CEikDocument`
- ⬜ `Reserved_2__9CEikAppUi`
- ⬜ `Reserved_3__9CEikAppUi`
- ⬜ `Reserved_4__9CEikAppUi`
- ⬜ `ResourceFileName__C15CEikApplication`
- ⬜ `RestoreL__12CEikDocumentRC12CStreamStoreRC17CStreamDictionary`
- ⬜ `RestoreMenuL__16MEikMenuObserverP11CCoeControliQ216MEikMenuObserver9TMenuType`
- ⬜ `SaveL__12CEikDocument`
- ⬜ `SaveL__12CEikDocumentQ213MSaveObserver9TSaveType`
- ⬜ `SetEmphasis__9CEikAppUiP11CCoeControli`
- ⬜ `StopDisplayingMenuBar__9CEikAppUi`
- ⬜ `StoreL__C12CEikDocumentR12CStreamStoreR17CStreamDictionary`
- ⬜ `UpdateTaskNameL__12CEikDocumentP19CApaWindowGroupName`
- ⬜ `ValidFileType__C9CEikAppUiG4TUid`
- ⬜ `_._12CEikDocument`
- ⬜ `_._15CEikApplication`
- ⬜ `__15CEikApplication`
- ⬜ `__9CEikAppUi`

## CONE (46)
_Control environment (window/control server client)._

- ⬜ `ActivateL__11CCoeControl`
- ⬜ `AddToStackL__9CCoeAppUiP11CCoeControlii`
- ⬜ `CONE_312`
- ⬜ `CONE_314`
- ⬜ `CONE_315`
- ⬜ `CONE_318`
- ⬜ `ComponentControl__C11CCoeControli`
- ⬜ `ConstructFromResourceL__11CCoeControlR15TResourceReader`
- ⬜ `CountComponentControls__C11CCoeControl`
- ⬜ `CreateWindowL__11CCoeControl`
- ⬜ `Draw__C11CCoeControlRC5TRect`
- ⬜ `DrawableWindow__C11CCoeControl`
- ⬜ `FocusChanged__11CCoeControl8TDrawNow`
- ⬜ `GetColorUseListL__C11CCoeControlRt9CArrayFix1Z12TCoeColorUse`
- ⬜ `GetHelpContext__C11CCoeControlR15TCoeHelpContext`
- ⬜ `HandleKeyEventL__9CCoeAppUiRC9TKeyEvent10TEventCode`
- ⬜ `HandlePointerBufferReadyL__11CCoeControl`
- ⬜ `HandlePointerEventL__11CCoeControlRC13TPointerEvent`
- ⬜ `HandleResourceChange__11CCoeControli`
- ⬜ `HandleSwitchOnEventL__9CCoeAppUiP11CCoeControl`
- ⬜ `HasBorder__C11CCoeControl`
- ⬜ `HelpContextL__C9CCoeAppUi`
- ⬜ `InputCapabilities__C11CCoeControl`
- ⬜ `InputCapabilities__C9CCoeAppUi`
- ⬜ `MCoeMessageObserver_Reserved_1__19MCoeMessageObserver`
- ⬜ `MCoeMessageObserver_Reserved_2__19MCoeMessageObserver`
- ⬜ `MCoeViewDeactivationObserver_Reserved_1__28MCoeViewDeactivationObserver`
- ⬜ `MCoeViewDeactivationObserver_Reserved_2__28MCoeViewDeactivationObserver`
- ⬜ `MakeVisible__11CCoeControli`
- ⬜ `MinimumSize__11CCoeControl`
- ⬜ `OfferKeyEventL__11CCoeControlRC9TKeyEvent10TEventCode`
- ⬜ `PositionChanged__11CCoeControl`
- ⬜ `PrepareForFocusGainL__11CCoeControl`
- ⬜ `PrepareForFocusLossL__11CCoeControl`
- ⬜ `RemoveFromStack__9CCoeAppUiP11CCoeControl`
- ⬜ `Reserved_2__11CCoeControl`
- ⬜ `SetAdjacent__11CCoeControli`
- ⬜ `SetAndDrawFocus__9CCoeAppUii`
- ⬜ `SetContainerWindowL__11CCoeControlRC11CCoeControl`
- ⬜ `SetDimmed__11CCoeControli`
- ⬜ `SetNeighbor__11CCoeControlP11CCoeControl`
- ⬜ `SizeChanged__11CCoeControl`
- ⬜ `SystemGc__C11CCoeControl`
- ⬜ `WriteInternalStateL__C11CCoeControlR12RWriteStream`
- ⬜ `_._11CCoeControl`
- ⬜ `__11CCoeControl`

## AVKON (19)
_S60 Avkon UI toolkit._

- ⬜ `BaseConstructL__CAknAppUii`
- ⬜ `ExecuteLD__CAknResourceNoteDialogRCTDesC16`
- ⬜ `HandleError__CAknAppUiiRCSExtendedErrorRTDes16T3`
- ⬜ `HandleForegroundEventL__CAknAppUii`
- ⬜ `HandleStatusPaneSizeChange__CAknAppUi`
- ⬜ `HandleSystemEventL__CAknAppUiRCTWsEvent`
- ⬜ `HandleViewDeactivation__CAknAppUiRCTVwsViewIdT1`
- ⬜ `HandleWsEventL__CAknAppUiRCTWsEventPCCoeControl`
- ⬜ `OpenFileL__CAknDocumentiRCTDesC16RRFs`
- ⬜ `OpenIniFileLC__CCAknApplicationRRFs`
- ⬜ `PreDocConstructL__CAknApplication`
- ⬜ `PrepareToExit__CAknAppUi`
- ⬜ `ProcessCommandL__CAknAppUii`
- ⬜ `Reserved_MtsmObject__CAknAppUi`
- ⬜ `Reserved_MtsmPosition__CAknAppUi`
- ⬜ `SetKeyBlockMode__CAknAppUiTAknKeyBlockMode`
- ⬜ `_._CAknAppUi`
- ⬜ `__CAknDocumentRCEikApplication`
- ⬜ `__CAknInformationNote`

## EFSRV (13)
_File server — asset I/O._

- ⬜ `Close__7RFsBase`
- ⬜ `Connect__3RFsi`
- ⬜ `Create__5RFileR3RFsRC7TDesC16Ui`
- ⬜ `Delete__3RFsRC7TDesC16`
- ⬜ `Flush__5RFile`
- ⬜ `MkDir__3RFsRC7TDesC16`
- ⬜ `Open__5RFileR3RFsRC7TDesC16Ui`
- ⬜ `Read__C5RFileR5TDes8`
- ⬜ `Read__C5RFileR5TDes8i`
- ⬜ `Seek__C5RFile5TSeekRi`
- ⬜ `SetSize__5RFilei`
- ⬜ `Size__C5RFileRi`
- ⬜ `Write__5RFileRC6TDesC8i`

## FBSCLI (7)
_Font & Bitmap server client._

- ⬜ `Create__10CFbsBitmapRC5TSize12TDisplayMode`
- ⬜ `DataAddress__C10CFbsBitmap`
- ⬜ `DisplayMode__C10CFbsBitmap`
- ⬜ `FBSCLI_156`
- ⬜ `Header__C10CFbsBitmap`
- ⬜ `SizeInPixels__C10CFbsBitmap`
- ⬜ `__10CFbsBitmap`

## BITGDI (6)
_Bitmapped graphics device interface (blitting)._

- ⬜ `Activate__9CFbsBitGcP10CFbsDevice`
- ⬜ `BITGDI_170`
- ⬜ `CreateContext__10CFbsDeviceRP9CFbsBitGc`
- ⬜ `SetDitherOrigin__9CFbsBitGcRC6TPoint`
- ⬜ `SetShadowMode__9CFbsBitGci`
- ⬜ `SetUserDisplayMode__9CFbsBitGc12TDisplayMode`

## APPARC (4)
_Application architecture (app bootstrap)._

- ⬜ `AppFullName__C15CApaApplication`
- ⬜ `Capability__C12CApaDocument`
- ⬜ `GlassPictureL__12CApaDocument`
- ⬜ `ValidatePasswordL__C12CApaDocument`

## ESTLIB (3)
_C standard library (P.I.P.S.-era)._

- ⬜ `sqrt`
- ⬜ `strcpy`
- ⬜ `strlen`

## MEDIACLIENTAUDIOSTREAM (1)
_Streaming audio output._

- ⬜ `CMdaAudioOutputStreamPadFunction__Fv`

## NOKIAFC (1)
_Nokia full-screen control (N-Gage framebuffer extension)._

- ⬜ `NOKIAFC_1`
