预览文档和呈现选项菜单

如果你的app需要打开它不支持的文件，请使用UIDocumentInteractionController。一个documentinteraction controller通过Quick Look框架判断文档是否能被另一个app打开和预览。也就是说，你的app通过documentinteraction controller提供给用户一些可选的操作。

要使用一个document interaction controller，需要以下步骤：

1.    针对每个你想打开的文件，创建相应的UIDocumentInteractionController实例。

2.     在app的UI中提供一个代表这个文件的图像化表示（一般，你应该显示文件名或者图标）。

3.    如果用户和这个对象发生交互，例如触摸了这个控件，则调用document interaction controller显示一个如下的界面：

o   预览文件的内容。

o   一个包含预览和打开操作的菜单。你可以通过实现某些委托方法，向菜单中加入其他操作，比如复制、打印。

o   一个菜单，仅包含“以其它方式打开”操作。

documentinteraction controller内置了一些手势，你可以直接实现它们的action。

如何需要与文件交互的app都可以使用document interaction controller。这些程序绝大部分都需要从网络下载文件。例如，email程序需要打开和预览邮件附件。

即使是从不下载文件的app，也需要document interaction controller。 例如 ，你的app需要支持文件共享（参考“File-Sharing Support” in iOS Technology Overview 以及DocInteraction 示例工程), 你可以对同步到app Documents/Shared目录下的文件使用documentinteraction controller。

创建DocumentInteraction Controller

要创建一个document interaction controller实例，用你想要打开的文件实例化它，并设置它的delegate属性。delegate对象负责告诉document  interaction controller呈现视图时需要的信息，以及当视图显示和用户交互时要执行的动作。

如以下代码所示。注意方法的调用者必须retain返回对象。

 

- (UIDocumentInteractionController *) setupControllerWithURL: (NSURL) fileURL
    usingDelegate: (id <UIDocumentInteractionControllerDelegate>) interactionDelegate {
 
    UIDocumentInteractionController *interactionController =
        [UIDocumentInteractionController interactionControllerWithURL: fileURL];
    interactionController.delegate = interactionDelegate;
 
    return interactionController;
}
 

创建好document interaction controller之后，可以通过它的属性来读取与之关联的文件信息，包括文件名、类型和 URL。该controllerｙ还有一个　icons 属性，其中包含了多个 UIImage 对象,本别用于表示该文档的多个大小的图标。这些信息可用于UI。

如果你想让用户用在其他程序中打开这个文件，可以使用controller的 annotation 属性，将该程序所需的附加信息放在其中。当然信息的格式必须能够被该应用程序识别。例如，该属性将被某个应用程序套件中的单个程序所用。当这个程序想与套件中的其他程序进行交互时，就可以使用annotation 属性。当调用应用程序打开一个文件时，option 字典中会包含 annotation 的值，可以使用UIApplicationLaunchOptionsAnnotationKey 作为键在option字典中检索它。

呈现 Document InteractionController

通过 Documentinteraction controller ，用户可以预览该文件，或者通过弹出菜单让用户选择相应的动作。

·      模式化显示文件预览窗口，调用 presentPreviewAnimated: 方法。

·      通过弹出菜单提示用户选择相应动作，调用presentOptionsMenuFromRect:inView:animated: 或者presentOptionsMenuFromBarButtonItem:animated: 方法。

·      提示用户用其他程序打开该文件，调用presentOpenInMenuFromRect:inView:animated: 方法或者presentOpenInMenuFromBarButtonItem:animated: 方法。

这些方法都会显示一个视图——要么是预览窗口，要么是弹出菜单。任何一个方法的调用，都要检查返回值。返回值为 NO，表示这个视图没有任何内容，将不能显示。例如，presentOpenInMenuFromRect:inView:animated:方法返回NO,表明已安装的程序中没有任何程序能够打开该文档。

如果你要显示预览窗口，委托对象必须实现 documentInteractionControllerViewControllerForPreview: 方法。预览窗口以模式窗口的形式显示，因此需要在该方法中返回一个view controller ，作为预览窗口的父窗口。如果你不实现该方法，或者在该方法中返回 nil，或者你返回的 view controller 无法呈现模式窗口，则该预览窗口不会显示。

document interactioncontroller 会自动解散它呈现出来的窗口。当然你也可以调用 dismissMenuAnimated: 或dismissPreviewAnimated: 方法手动解散它 。

在示例项目 DocInteraction 中，演示了如何呈现一个document interaction controller（使用了手势识别）。

 

注册应用程序支持的文档类型 

如果你的程序可以打开某种特定的文件类型，则你可以通过Info.pllist 文件注册程序所能打开的文档类型。当其他程序向系统询问哪些程序可以识别该类型的文件时，你的程序将会被列到选项菜单中，供用户选择。

相关章节: “注册应用程序支持的文件类型”

注册应用程序支持的文件类型

如果你的程序能够打开某种文件，你可以向系统进行注册。这会运行其他程序通过 iOS 的 document interaction 技术提供给用户一个选择，从而调用你的程序处理这些文件。

这需要在程序的Info.plist 文件中包含 CFBundleDocumentTypes 键 (查看 “CoreFoundation Keys”) 。系统将该键中包含的内容进行登记，这样其他程序就可以通过 document interaction controller 访问到这些信息。

CFBundleDocumentTypes 键是一个dictionary数组，一个dictionary表示了一个指定的文档类型。一个文档类型通常与某种文件类型是一一对应的。但是，如果你的程序对多个文件类型采用同样的处理方式，你也可以把这些类型都分成一个组，统一视作一个文档类型。例如，你的程序中使用到的本地文档类型，有一个是旧格式的，还有一个新格式（似乎是影射微软office文档），则你可以将二者分成一组，都放到同一个文档类型下。这样，旧格式和新格式的文件都将显示为同一个文档类型，并以同样的方式打开。

CFBundleDocumentTypes 数组中的每个dictionary 可能包含以下键： 

·      CFBundleTypeName 指定文档类型名称。

·      CFBundleTypeIconFiles 是一个数组，包含多个图片文件名，用于作为该文档的图标。

·      LSItemContentTypes 是一个数组，包含多个 UTI 类型的字符串。UTI 类型是本文档类型（组）所包含的文件类型。

·      LSHandlerRank 表示应用程序是“拥有”还是仅仅是“打开”这种类型而已。

在应用程序的角度而言，一个文档类型其实就是一种文件类型（或者多个文件类型），该程序将一个文档类型的文件都视作同样的东西对待。例如，一个图片处理程序可能将各种图片文件都看成不同的文档类型，这样便于根据每个类型进行相应的优化。但是，对于字处理程序来说，它并不关心真正的图形格式，它把所有的图片格式都作为一个文档类型对待。

下表列出了Info.plist 中的一个 CFBundleTypeName 示例。

LSItemContentTypes 表示与文件格式对应的 UTI，CFBundleTypeIconFiles 指定了该类型所用的图标资源。

表 1  自定义文件格式的文档类型 

<dict>
   <key>CFBundleTypeName</key>
   <string>My File Format</string>
   <key>CFBundleTypeIconFiles</key>
       <array>
           <string>MySmallIcon.png</string>
           <string>MyLargeIcon.png</string>
       </array>
   <key>LSItemContentTypes</key>
       <array>
           <string>com.example.myformat</string>
       </array>
   <key>LSHandlerRank</key>
   <string>Owner</string>
</dict>
关于CFBundleDocumentTypes 键的更多信息, 可参考 InfoPlist 键参考。

 

在其他应用程序中打开文件

系统可能会请求某个程序打开某种文件，并呈现打开选项菜单给用户。最为常见的情况是，某个应用程序遇到了你的程序注册了支持的文件类型。这样，系统会将文件的 URL 传递给你的应用程序，并将它放入前台。

相关章节: “打开支持的文件类型”

打开支持的文件类型

系统可能会请求某个程序打开某种文件，并呈现给用户。通常这发生在某个应用程序调用 document interaction controller 去处理某个文件的时候。你可以在应用程序委托的

 application:didFinishLaunchingWithOptions: 方法中获得该文件的信息。如果你的程序要处理某些自定义的文件类型，你必须实现这个委托方法（而不是applicationDidFinishLaunching:  方法) 并用这个方法启动应用程序。

 

application:didFinishLaunchingWithOptions: 方法的 option 参数包含了要打开的文件的相关信息。尤其需要在程序中关心下列键：

·      UIApplicationLaunchOptionsURLKey 包含了该文件的NSURL.

·      UIApplicationLaunchOptionsSourceApplicationKey 包含了发送请求的应用程序的 Bundle ID。

·      UIApplicationLaunchOptionsSourceApplicationKey 包含了源程序向目标程序传递的与该文件相关的属性列表对象。

如果 UIApplicationLaunchOptionsURLKey 键存在，你的程序应当立即用该 URL 打开该文件并将内容呈现给用户。其他键可用于收集与打开的文件相关的参数和信息。

 

Quick Look Previews 中的预览及打印

要想获得更多的文件预览功能，你可以直接使用 Quick Look 框架。你可以选择呈现预览窗口时的动画风格，并可以想预览单个文件一样预览多个文件。QuickLook preview controller 还内置了所支持的文件类型的 AirPrint 打印。

相关章节: “使用 Quick Look 框架”

使用 Quick Look 框架

 

Quick Look 框架提供了增强的预览功能。该框架主要提供了 QLPreviewController 类。该类依赖于委托对象响应预览动作，以及一个用于提供预览文件的数据源。

从 iOS 4.2 开始，QuickLook preview controller 提供了包含了一个 action 按钮（即打印按钮）的预览视图。对于 controller 能预览的文件，action按钮能够打印该文档。从而不需要你编写任何打印代码。

通过以下方式显示一个Quick Look preview controller:

·      通过导航控制器，将预览窗口以“push 方式”显示。

·      通过 UIViewController 的 presentModalViewController:animated:方法以模态窗口的方式显示。

·      显示一个document interaction controller(如 “预览及打开文件” 中所述）。用户可以从document interaction controller 的选项菜单中选择“Quick Look”，即可打开一个 QuickLook preview controller。

显示 Quick Lookpreview controller 时，请选择适合于你的应用程序的外观和导航方式。如果你的程序未使用导航条，使用一个全屏的模态的 Quick Lookpreview controller 是合适的。如果你的程序使用了“iPhone 式”的导航，则采用 push 方式呈现预览窗口是合适的。

预览窗口中会包括一个标题，显示文件 URL 的最后一段路径。如果要重载标题，可以定制PreviewItem 类，并实现QLPreviewItem 协议中的 previewItemTitle方法。

Quick Look previewcontroller 能够预览下列文件：

·      iWork 文档

·      Microsoft Office 文档(Office ‘97 以后版本)

·      Rich Text Format (RTF) 文档

·      PDF 文档

·      图片

·      文本文件，其 uniform type identifier (UTI)  在 public.text 文件中定义 (查看UniformType Identifiers 参考)

·      Comma-separated value (csv) 文件

使用 QuickLook preview controller，必须指定数据源对象（即实现 QLPreviewControllerDataSource 协议，请查看QLPreviewControllerDataSource协议参考）。数据源为 Quick Look preview controller 提供预览对象（preivew item），及指明它们的数量以便在一个预览导航列表中包含它们。在这个列表中包含多个对象，在模态预览窗口（全屏显示）显示了导航箭头，以便用户在多个预览对象间切换。对于用导航控制器“push方式”显示的QuickLook preview controller，你可以在导航条上提供按钮以便在预览对象列表见切换。

Quick Look framework 的具体介绍，见 iOSQuick Look 框架手册.

 

 

#import "MTViewController.h" 

@interface MTViewController () <UIDocumentInteractionControllerDelegate>

@property (strong, nonatomic) UIDocumentInteractionController *documentInteractionController;

 

@end

 

@implementation MTViewController

 

#pragma mark -

#pragma mark View Life Cycle

- (void)viewDidLoad {

    [super viewDidLoad];

}

 

#pragma mark -

#pragma mark Actions

- (IBAction)previewDocument:(id)sender { // 预览

    NSURL *URL = [[NSBundle mainBundle] URLForResource:@"sample" withExtension:@"pdf"];

    

    if (URL) {

        // Initialize Document Interaction Controller

        self.documentInteractionController = [UIDocumentInteractionController interactionControllerWithURL:URL];

        

        // Configure Document Interaction Controller

        [self.documentInteractionController setDelegate:self];

        

        // Preview PDF

        [self.documentInteractionController presentPreviewAnimated:YES];

    }

}

 

- (IBAction)openDocument:(id)sender {　　// 打开

    UIButton *button = (UIButton *)sender;

    NSURL *URL = [[NSBundle mainBundle] URLForResource:@"sample" withExtension:@"pdf"];

    

    if (URL) {

        // Initialize Document Interaction Controller

        self.documentInteractionController = [UIDocumentInteractionController interactionControllerWithURL:URL];

        

        // Configure Document Interaction Controller

        [self.documentInteractionController setDelegate:self];

        

        // Present Open In Menu

        [self.documentInteractionController presentOpenInMenuFromRect:[button frame] inView:self.view animated:YES];

    }

}

 

#pragma mark -

#pragma mark Document Interaction Controller Delegate Methods

- (UIViewController *) documentInteractionControllerViewControllerForPreview: (UIDocumentInteractionController *) controller {

    return self;

}

 

@end

test  什么时候可以发现

