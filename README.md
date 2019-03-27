# Silkroad Online Second Password System

## Story
A while ago I started to look at the second password system at iSRO (shortly passcode system) to know how it work,
I found that when the user enter the passcode the client encrypt and send it to the server encryptly with the passcode length,
And this was a try to stop bots from making auto-login system, So the bot now got two options, first is to disable auto-login
and make it manual-login, And the second is to make manual-login at first time so the bot can get the encrypted passcode
then use it to re-login again and again and ... again.

After a lot of RE work i found that the passcode was encrypted by Silkroad Blowfish that was created by 'pushedx'
as a part of Silkroad Security Api but with a different key, So let's dig in! and start coding.

## ASM Look
When you try to trace the passcode encryption progress it's a good idea to start from the Click Button Event
and trace a string like 'The passcode must be between 6-8 digits.', If you trace it right your debugger should land here:
(Client Version: 1.567)

```
00494E25 | 33C0                      | xor eax,eax                                                |
00494E27 | 8D4C24 10                 | lea ecx,dword ptr ss:[esp+10]                              |
00494E2B | C64424 24 00              | mov byte ptr ss:[esp+24],0                                 |
00494E30 | 894424 25                 | mov dword ptr ss:[esp+25],eax                              |
00494E34 | 66:894424 29              | mov word ptr ss:[esp+29],ax                                |
00494E39 | 884424 2B                 | mov byte ptr ss:[esp+2B],al                                |
00494E3D | C64424 1C 0F              | mov byte ptr ss:[esp+1C],F                                 | ;Key[0]
00494E42 | C64424 1D 07              | mov byte ptr ss:[esp+1D],7                                 | ;Key[1]
00494E47 | C64424 1E 3D              | mov byte ptr ss:[esp+1E],3D                                | ;Key[2]
00494E4C | C64424 1F 20              | mov byte ptr ss:[esp+1F],20                                | ;Key[3]
00494E51 | C64424 20 56              | mov byte ptr ss:[esp+20],56                                | ;Key[4]
00494E56 | C64424 21 62              | mov byte ptr ss:[esp+21],62                                | ;Key[5]
00494E5B | C64424 22 C9              | mov byte ptr ss:[esp+22],C9                                | ;Key[6]
00494E60 | C64424 23 EB              | mov byte ptr ss:[esp+23],EB                                | ;Key[7]
00494E65 | E8 A63AFDFF               | call <sro_client.Blowfish::ctor()>                         |
00494E6A | C74424 38 00000000        | mov dword ptr ss:[esp+38],0                                |
00494E72 | 6A 08                     | push 8                                                     | ;KeyLength
00494E74 | 8D4C24 20                 | lea ecx,dword ptr ss:[esp+20]                              |
00494E78 | 51                        | push ecx                                                   | ;KeyPointer
00494E79 | 8D4C24 18                 | lea ecx,dword ptr ss:[esp+18]                              |
00494E7D | E8 7E42FDFF               | call <sro_client.Blowfish::Initialize(key_ptr, key_size)>  |
00494E82 | 837E 18 10                | cmp dword ptr ds:[esi+18],10                               |
00494E86 | 72 05                     | jb sro_client.494E8D                                       |
00494E88 | 8B4E 04                   | mov ecx,dword ptr ds:[esi+4]                               |
00494E8B | EB 03                     | jmp sro_client.494E90                                      |
00494E8D | 8D4E 04                   | lea ecx,dword ptr ds:[esi+4]                               | ;Pin
00494E90 | 55                        | push ebp                                                   | ;PinLength
00494E91 | 8D5424 14                 | lea edx,dword ptr ss:[esp+14]                              |
00494E95 | 52                        | push edx                                                   | ;PinPointer
00494E96 | 8D4424 2C                 | lea eax,dword ptr ss:[esp+2C]                              |
00494E9A | E8 7146FDFF               | call <sro_client.Blowfish::Encode(input_ptr, input_size)>  |
```

I added some comments to make it easy to understand, From this 
Let's convert it to C++ / C# Code.

# Codes

```csharp
Blowfish blowfish = new Blowfish();
byte[] key = { 0x0F, 0x07, 0x3D, 0x20, 0x56, 0x62, 0xC9, 0xEB };
blowfish.Initialize(key, 0, key.Length);

byte[] encodedPin = blowfish.Encode(Encoding.ASCII.GetBytes(pin));
```
