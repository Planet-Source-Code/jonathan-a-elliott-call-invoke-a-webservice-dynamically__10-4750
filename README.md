<div align="center">

## Call/Invoke a Webservice Dynamically


</div>

### Description

I found a way after searching the web to invoke a webservice thru code without having to set a web reference.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Jonathan A\. Elliott](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/jonathan-a-elliott.md)
**Level**          |Intermediate
**User Rating**    |5.0 (15 globes from 3 users)
**Compatibility**  |C\#
**Category**       |[Server Side](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/server-side__10-31.md)
**World**          |[\.Net \(C\#, VB\.net\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/net-c-vb-net.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/jonathan-a-elliott-call-invoke-a-webservice-dynamically__10-4750/archive/master.zip)





### Source Code

internal class WsProxy
 {
  [SecurityPermissionAttribute(SecurityAction.Demand, Unrestricted = true)]<BR>
  internal static object CallWebService(string webServiceAsmxUrl, string serviceName, string methodName, object[] args)<BR>
  {<BR>
   System.Net.WebClient client = new System.Net.WebClient();<BR>
   //-Connect To the web service<BR>
   System.IO.Stream stream = client.OpenRead(webServiceAsmxUrl + "?wsdl");<BR>
   //--Now read the WSDL file describing a service.<BR>
   ServiceDescription description = ServiceDescription.Read(stream);<BR>
   ///// LOAD THE DOM /////////<BR>
   //--Initialize a service description importer.<BR>
   ServiceDescriptionImporter importer = new ServiceDescriptionImporter();<BR>
   importer.ProtocolName = "Soap12"; // Use SOAP 1.2.<BR>
   importer.AddServiceDescription(description, null, null);<BR>
   //--Generate a proxy client.
   importer.Style = ServiceDescriptionImportStyle.Client;<BR>
   //--Generate properties to represent primitive values.<BR>
   importer.CodeGenerationOptions = System.Xml.Serialization.CodeGenerationOptions.GenerateProperties;<BR>
   //--Initialize a Code-DOM tree into which we will import the service.<BR>
   CodeNamespace nmspace = new CodeNamespace();<BR>
   CodeCompileUnit unit1 = new CodeCompileUnit();<BR>
   unit1.Namespaces.Add(nmspace);<BR>
   //--Import the service into the Code-DOM tree. This creates proxy code<BR>
   //--that uses the service.<BR>
   ServiceDescriptionImportWarnings warning = importer.Import(nmspace, unit1);<BR>
   if (warning == 0) //--If zero then we are good to go<BR>
   {<BR>
    //--Generate the proxy code
    CodeDomProvider provider1 = CodeDomProvider.CreateProvider("CSharp");<BR>
    //--Compile the assembly proxy with the appropriate references<BR>
    string[] assemblyReferences = new string[5] { "System.dll", "System.Web.Services.dll", "System.Web.dll", "System.Xml.dll", "System.Data.dll" };<BR>
    CompilerParameters parms = new CompilerParameters(assemblyReferences);<BR>
    CompilerResults results = provider1.CompileAssemblyFromDom(parms, unit1);<BR>
    //-Check For Errors<BR>
    if (results.Errors.Count > 0)<BR>
    {<BR>
     foreach (CompilerError oops in results.Errors)<BR>
     {<BR>
      System.Diagnostics.Debug.WriteLine("========Compiler error============");<BR>
      System.Diagnostics.Debug.WriteLine(oops.ErrorText);<BR>
     }<BR>
     throw new System.Exception("Compile Error Occured calling webservice. Check Debug ouput window.");<BR>
    }<BR>
    //--Finally, Invoke the web service method
    object wsvcClass = results.CompiledAssembly.CreateInstance(serviceName);<BR>
    MethodInfo mi = wsvcClass.GetType().GetMethod(methodName);<BR>
    return mi.Invoke(wsvcClass, args);<BR>
   }<BR>
   else<BR>
   {<BR>
    return null;<BR>
   }<BR>
  }<BR>
 }<BR>

