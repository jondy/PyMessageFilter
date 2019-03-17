# PyMessageFilter

A python package implements interface IMessageFilter for COM.

The main purpose is to fix 'Application is Busy' and 'Call was Rejected By Callee' errors in a WINDOWS COM/COM+ client. Here is a workaround

https://docs.microsoft.com/en-us/previous-versions/ms228772(v=vs.140)

But it's written by C#, this package offers both C and Python versions.

## Usage

Install it by `pip install PyMessageFilter`

Import `CMessageFilter` from `PyMessageFilter`

Call static method `register` at the beginning of automation

Call static method `revoke` at the end of work

Here it's an example

```python
from comtypes.client import CreateObject
from PyMessageFilter import CMessageFilter

def iter_excel_sheets(filename):
    CMessageFilter.register()
    aobj = CreateObject('Excel.Application', dynamic=True)
    aobj.Workbooks.Open(filename)
    for sheet in aobj.ActiveWorkbook.Worksheets:
        print('Processing %s' % sheet.Name)
    CMessageFilter.revoke()
```

Note that interface `IMessageFilter` only works in single-threaded apartments, it will not be called in the multithreaded apartments. Refer to

https://docs.microsoft.com/en-us/windows/desktop/com/choosing-the-threading-model

### Build `msgfilter.dll`

By visual studio command line `cl /LD msgfilter.c ole32.lib`

By mingw32 in cygwin `i686-pc-mingw32-gcc -shared -o msgfilter.dll msgfilter.c -lole32 -luuid`

There are 2 export functions, they can be used in any c file:

```c
extern void register_message_filter();
extern void revoke_message_filter();

int main(int argc, char* argv[])
{
  HRESULT ret;
  ret = CoInitializeEx(0, COINIT_APARTMENTTHREADED);
  register_message_filter();

  // do something for automation

  revoke_message_filter();
  CoUninitialize();

  return 0;
}

```
