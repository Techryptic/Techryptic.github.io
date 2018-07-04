---
layout:     post
title:      "Google Capture The Flag 2018"
#subtitle:   "ctf"
date:       2018-06-27
author:     "Tech"
header-img: "img/post-googlectf18.png"
tags:
    - ctf
---

![](/img/in-post/post-js-version/googlectf-letter.png)

# LETTER Challenge:

For this challenge, you'll need to rename the file to a .pdf and open it with any pdf viewer. Copy the blacked-out text and paste it into any text editor. The flag is in the password field.

`CTF{ICanReadDis}`

---
---

![](/img/in-post/post-js-version/googlectf-moar.png)

# MOAR Challenge:
Socat serving man page of socat with less.  Use ! to execute commands.  Can’t `!/bin/sh` but can do things like `!whoami` (moar)  `!cat /home/moar/*`
#!/bin/sh echo 'Disabling DMZ using password CTF{SOmething-CATastr0phic}' echo CTF{SOmething-CATastr0phic} > /dev/dmz

`CTF{S0mething-CATastr0phic}`

---
---

![](/img/in-post/post-js-version/googlectf-admin.png)

# Admin UI Challenge:
Restricted shell - first function is blocked by password, and second has directory traversal.

You select  ```../../../proc/self/environ```  ```../../../proc/self/cmdline```  running program is ```./main  ../main``` returns the file.  Extract with Wireshark.  Look at it in IDA.  `void __cdecl primary_login…`


Opening it in IDA, when we go to the function :

```c
void __cdecl secondary_login()
{
  bool v0; // al@6
  char password[128]; // [rsp+0h] [rbp-90h]@1
  size_t l; // [rsp+80h] [rbp-10h]@1
  size_t i; // [rsp+88h] [rbp-8h]@1

  puts("! Two factor authentication required !");
  puts("Please enter secret secondary password:");
  scanf("%127s", password);
  l = strlen(password);
  for ( i = 0LL; i < l; ++i )
    password[i] ^= 0xC7u;
  v0 = 0;
  if ( l == 35 )
  {
    *(_QWORD *)password = *(_QWORD *)FLAG;
    *(_QWORD *)&password[8] = *(_QWORD *)&FLAG[8];
    *(_QWORD *)&password[16] = *(_QWORD *)&FLAG[16];
    *(_QWORD *)&password[24] = *(_QWORD *)&FLAG[24];
    *(_WORD *)&password[32] = *(_WORD *)&FLAG[32];
    password[34] = FLAG[34];
    if ( password )
      v0 = 1;
  }
  if ( !v0 )
  {
    puts("Access denied.");
    exit(1);
  }
  puts("Authenticated");
  command_line();
}
```

From here we can see a few things:
* Password must be 35 characters long
 * Password XOR with ```0xC7```
* FLAG is in binary at ```41414A40```

I like to use cyberchef when working on CTFs, from here we can get the flag:

>  [https://gchq.github.io/CyberChef/#recipe=From_Hex('Space')XOR(%7B'option':'Hex','string':'C7'%7D,'Standard',false)&input=IDg0IDkzIDgxIEJDIDkzIEIwIEE4IDk4ICA5NyBBNiBCNCA5NCBCMCBBOCBCNSA4MwogQkQgOTggODUgQTIgQjMgQjMgQTIgQjUgIDk4IEIzIEFGIEYzIEE5IDk4IEY2IDk4CiBBQyBGOCBCQSA](https://gchq.github.io/CyberChef/#recipe=From_Hex('Space')XOR(%7B'option':'Hex','string':'C7'%7D,'Standard',false)&input=IDg0IDkzIDgxIEJDIDkzIEIwIEE4IDk4ICA5NyBBNiBCNCA5NCBCMCBBOCBCNSA4MwogQkQgOTggODUgQTIgQjMgQjMgQTIgQjUgIDk4IEIzIEFGIEYzIEE5IDk4IEY2IDk4CiBBQyBGOCBCQSA)

`CTF{Two_PasSworDz_Better_th4n_1_k?}`

---
---

![](/img/in-post/post-js-version/googlectf-floppy.png)

# FLOPPY Challenge:
If you use `xxd` on foo.ico you see that there is a zip file in there (PK magic number).  Use `foremost` to carve and open the zip. Inside is a driver.txt file with the flag:

`CTF{qeY80sU6Ktko8BJW}`

---
---

![](/img/in-post/post-js-version/googlectf-ocr.png)

# OCR IS COOL
Extract the png, which looks like to be a substitution cypher.

Here's the extracted text:
```
Wxtk Vnlmhfxk,
Px tkx atiir mh pxevhfx tl hnk gxpxlm vnlmhfxk hy hnx lxvnkx bWkhiWkbox vehnw ybex latkbgz lxkobvx.  T ltyx ietvx yhk tee rhnk ybexl.Lmhkx tgr ybex
bWkhiWkbox lmtkml rhn pbma 15 IU hy ykxx hgebgx hk hyyebgx lmhktzx, lh rhn vtg dxxi ptkxs, ubgtkbxl, itbgmbgzl, yetzl, ybkfptkxl, ubmvhbgl, pkbmxnil - tgrmabgz.Lxx rhnk lmnyy tgrpaxkx
Rhnk ybexl bg bWkhiWkbox vtg ux kxtvaxw ykhf tgr yhhutgbsxk, lftkm ykbwzx 2000, lftkm atnl, Mxfih-t-ftmbv, hk fxwbt iv. Lh paxkxoxk rhn zh, rhnk ybexl yheehp.Latkx ybexl tgw yhewxkl
Rhn vtg jnbvder bgobmx hmaxkl mh obxp, whpgehtw, tgw vheetuhktmx hg tee max ybexl rhn ptgm obt xftbe tmmtvafxgml. Cnlm zbox maxf max ebgd VMY{vtxltkvbiaxkbltlnulmbmnmbhgvbiaxk} tgw maxr vtg tvvxll tee rhnk wtmt.Yhk xqtfiex, axkx'l t eblm hy ybexl matm rhn'kx vnkkxgmer lmhkbgz pbma nl:
hyyanu_ybkfptkx.ubg ChagMK
BHM_vkxwxgmbtel.iwy\ \(wxexmxw\) Pbgmxkfnmxw
Yhhutgbsxk9000_Ftgnte.iwy Pbgmxkfnmxw
yhh.bvh Mnkuh
Lbgvx px mtdx lxvnkbmr oxkr lxkbhnler tgw bg hkwxk mh ikhmxvm rhn tztbglm onegxktubebmbxl ebdx xytbe tgw tfwyetpl, px'kx lxgwbgz rhn rhnk vkxwxgmbtel nlbgz max mbfx-ikhoxg fbebmtkr-zktwx vtxltk lrffxmkbv vbiaxkAtiir bWkhiWkbobgz!

```
Here's a quick python script to decode the text:
```python
#!/usr/bin/python3
f = open("substitution.txt")
orig_text=f.read()
new_text=""
i = 0
while i < len(orig_text):
  orig_ch = orig_text[i]
  orig_int = ord(orig_ch)
  if (orig_int > 64 and orig_int < 91):
    if (orig_int > 83 ):
      new_int = orig_int - 19
    else:
      new_int = orig_int + 7
  elif (orig_int > 96 and orig_int < 123):
    if (orig_int > 115):
      new_int = orig_int - 19
    else:
      new_int = orig_int + 7
  else:
    new_int = orig_int
  new_ch = chr(new_int)
  new_text += (new_ch)
  i += 1
print(new_text)
```

After you decode it, flag is:

`CTF{caesarcipherisasubstitutioncipher}`

---
---

![](/img/in-post/post-js-version/googlectf-obscurity.png)

# Security by Obscurity
Using `Johntheripper`, Grabbed & Compiled source (debian's didn't seem to have zip functionality). Following instructions to get the hash from the zip and then running john against the hash cracked the password to 'asdf'. Extracting with password gives key:

`CTF{CompressionIsNotEncryption}`

---
---

![](/img/in-post/post-js-version/googlectf-firmware.png)

# Firmware
Download the attached zip file.  Extracting gives it as an `ext4` extension.  Mounting the file shows a file ```'.mediapc_backdoor_password.gz'``` in root of mounted filesystem.  Gunzip'ing it (zcat would have worked I guess) outputs text file with key:

`CTF{I_kn0W_tH15_Fs}`

---
---

![](/img/in-post/post-js-version/googlectf-gatekeeper.png)

# GATEKEEPER Challenge: (Reverse Engineering)
A zip, of course; inspecting the extracted file, looks to be a binary. `chmod+x` & run binary. simple program, wants username & password.  inspecting output of `'objdump'` finds password "0n3_W4rM" and possible password "zLl1ks_d4m_T0g_I"  -- stuck for a long time. Password is reversed:  "I_g0T_m4d_sk1lLz"

`CTF{I_g0T_m4d_sk1lLz}`

---
---
