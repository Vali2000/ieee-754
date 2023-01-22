# ieee-754

unit uIEEE754;

interface

uses
  Windows, Messages, SysUtils, Classes, Graphics, Controls, Forms, Dialogs,
  StdCtrls, Math;

type
  TfIEEE754 = class(TForm)
    Edit1: TEdit;
    Edit2: TEdit;
    Edit3: TEdit;
    Edit4: TEdit;
    Button1: TButton;
    Button2: TButton;
    Button3: TButton;
    Label1: TLabel;
    Label2: TLabel;
    Label3: TLabel;
    Label4: TLabel;
    procedure Button1Click(Sender: TObject);
    procedure Button2Click(Sender: TObject);
    procedure Button3Click(Sender: TObject);
    procedure FormCreate(Sender: TObject);
  private
    //function HToI(aStr: String): Integer;
    function SToI(aStr: String): Integer;
    function SToR(aStr: String): Extended;
    function DigitToChar(digit: Byte): String;
    //function BToSBit(d: Byte): String;
    function Dec2Bin(iDec: Integer): String;
    function SBitToInt(s: String): Integer;
    function ConvSIntToSExt(param: String): String;
    //function ConvSExtToSInt(param: String): String;
    //function ConvSExtToSInt2(param: String): String;
    function ConvSExtToSInt(param: String): String;
  public

  end;

var
  fIEEE754: TfIEEE754;

implementation

{$R *.DFM}

(*
function TForm1.HToI(aStr: String): Integer;
const Digits: Array[0..$F] of Char = '0123456789ABCDEF';
var i, j: Integer;
begin
  Result:= 0;
  for i:=1 to Length(aStr) do
    if aStr[i] in ['0'..'9','a'..'f','A'..'F'{,'-'}]
    then begin
      j:= Pos(UpperCase(aStr[i]),Digits) - 1;
      if j > -1 then
        Result:= Result * 16 + j;
    end;
end;//HToI
*)

function TfIEEE754.SToI(aStr: String): Integer;
var v, sign, i: Integer;
begin
  v:= 0;
  sign:= 1;
  for i:=1 to Length(aStr) do
    if (aStr[i] >= '0') and (aStr[i] <= '9') then
      v:= v * 10 + Ord(aStr[i]) - $30
    else if (aStr[i] = '-') and (sign = 1) then
      sign:= -1;
  Result:= sign * v;
end;//SToI

function TfIEEE754.SToR(aStr: String): Extended;
var iv, fv: Extended;
    l, i, j, k, sign: Integer;
begin
  iv:= 0;
  fv:= 0;
  k:= 0;
  sign:= 1;
  l:= Length(aStr);
  for i:=1 to l do begin
    if k = 0 then
      j:= i
    else
      j:= l + k - i;
    if (aStr[j] >= '0') and (aStr[j] <= '9') then
      if k = 0 then
        iv:= iv * 10 + Ord(aStr[j]) - $30
      else
        fv:= fv / 10 + Ord(aStr[j]) - $30
    else
      if (aStr[j] = '-') and (sign = 1) then
        sign:= -1
    else
      if (aStr[j] = '.') and (k = 0) then
        k:= j + 1
  end;
  Result:= sign * (iv + fv / 10);
end;//SToR

function TfIEEE754.DigitToChar(digit: Byte): String;
begin
  if digit in [0..9] then
    DigitToChar:= Chr(digit+Ord('0')) //Format('%.1d',[digit])
  else
    DigitToChar:='0'
end;//DigitToChar

{function TForm1.BToSBit(d: Byte): String;
const mask: Array[1..8] of Byte = ($80,$40,$20,$10,$08,$04,$02,$01);

var bit, i: Byte; s: String;
begin
  for i:=1 to 8 do begin
    bit:= (d and mask[i]) shr (8-i);
    s:= s + DigitToChar(bit);
  end;
  result:= s;
end;}

function TfIEEE754.SBitToInt(s: String): Integer;
var i, r: Integer;
begin
  r:= 0;
  for i:=1 to Length(s) do
    r:= r * 2 + Ord(s[i]) - 48;
  Result:= r;
end;

function TfIEEE754.Dec2Bin(iDec: Integer): String;
begin
  Result:='';
  while iDec > 0 do begin
    Result:= IntToStr(iDec and 1)+Result;
    iDec:= iDec shr 1;
  end;
end;

function TfIEEE754.ConvSIntToSExt(param: String): String;
var
  ipar, sign, expo, mant, bman, iman: Integer;
  fmant, fpar: Extended;
begin
  ipar:= SToI(param);
  sign:= ipar shr 31; if sign=0 then sign:= 1 else sign:= -1;
  expo:= ((ipar shr 23) and $FF) - 127;
  mant:= ipar and $7FFFFF;
  fmant:= 0;
  for iman:=0 to 22 do begin
    bman := (mant shr iman) and 1;
    fmant:= fmant + bman * Power(2,iman-23);
  end;
  fmant:= 1 + fmant;
  fpar:= sign * Power(2, expo) * fmant;
  Result:= Trim(Format('%15.4f',[fpar]));
end;//ConvSIntToSExt

// documentatie:
// https://class.ece.iastate.edu/arun/Cpre305/ieee754/ie4.html
// https://www.binaryconvert.com/
function TfIEEE754.ConvSExtToSInt(param: String): String;
var
  ipar, iexpo, i: Integer;
  ssign, sexpo, smant, spar: String;
  imant1, imant2, savefmant, fmant, fpar: Extended;
begin
  Result:= '';
  fpar:= SToR(param);
  if fpar = 0 then begin
    ipar:= 0;
    Result:= Format('%d',[ipar]);
    Exit;
  end;//fpar = 0
  //1. sign
  if fpar < 0 then begin
    fpar:= -fpar;
    ssign:= '1';
  end
  else begin
    ssign:= '0';
  end;
  //2. exponent
  i:=0;
  fmant:= 0;
  if fpar < 2 then begin
    while True do begin
      fmant:= fpar / Power(2,i);
      if fmant >= 1 then break; // PRIMA VALOARE >= 1 !!!
      i:= i - 1;
    end//while True
  end//fpar < 2
  else begin//fpar >= 2
    while True do begin
      savefmant:= fmant;
      fmant:= fpar / Power(2,i);
      if fmant < 1 then begin
        fmant:= savefmant;   // ULTIMA VALOARE >= 1 !!!
        i:= i - 1;
        break;
      end;//fmant < 1
      i:= i + 1;
    end;//while True
  end;//fpar >= 2
  iexpo:= i;
  iexpo:= iexpo + 127;
  sexpo:= Dec2Bin(iexpo);
  //umple cu 0 la stanga pana la lungimea 8
  sexpo:= StringOfChar('0',8-Length(sexpo)) + sexpo;
  //3. mantissa fraction
  fmant:= fmant - 1;
  imant1:= 0;
  for i:=1 to 23 do begin
    imant2:= imant1 + Power(2,-i);
    if imant2 <= fmant then begin
      imant1:= imant2;
      smant:= smant + '1';
    end
    else begin
      smant:= smant + '0';
    end;
  end;
  spar:= ssign + sexpo + smant;
  ipar:= SBitToInt(spar);
  Result:= Format('%d',[ipar]);
end;//ConvSExtToSInt

procedure TfIEEE754.Button1Click(Sender: TObject);
begin
  Edit2.Text:= ConvSIntToSExt(Edit1.Text);
end;

procedure TfIEEE754.Button2Click(Sender: TObject);
begin
  Edit4.Text:= ConvSExtToSInt(Edit3.Text);
end;

procedure TfIEEE754.Button3Click(Sender: TObject);
begin
  Edit1.Text:= Edit4.Text;
  Button1Click(nil);
end;

procedure TfIEEE754.FormCreate(Sender: TObject);
begin
  Button1Click(nil);
  Button2Click(nil);
end;

end.

